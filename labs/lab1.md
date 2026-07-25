# Lab1 Notes

## 1. Circuit Layers & Parallelism

- Two gates can share a layer only if they touch **disjoint qubits**.
- A qubit involved in a gate can't be reused in another gate in the same layer.
- **Broadcast/fan-out time (GHZ prep from a center qubit):** if a qubit has *k* branches, it needs at least *k* rounds just to inform them all — one call per round. The bottleneck is whichever branch is called last, plus however many more hops that branch needs to reach its leaf.
- **BFS gives the minimum layer count for any connectivity graph:**
  ```python
  parent = {center: None}
  bfs_depth = {center: 0}
  queue = deque([center])
  while queue:
      node = queue.popleft()
      for neighbor in adjacency[node]:
          if neighbor not in parent:
              parent[neighbor] = node
              bfs_depth[neighbor] = bfs_depth[node] + 1
              queue.append(neighbor)
  ```


## 2. Transpilation

- Converts an abstract circuit into one that respects a specific device's constraints:
  - **Native gate set** (e.g. `RZ`, `SX`, `CZ` on IBM hardware) — anything else gets decomposed.
  - **Limited qubit connectivity** — two-qubit gates only work between physically adjacent qubits.
- `transpile(qc, backend=backend, optimization_level=3)` — optimization_level (0–3) trades transpile time for how hard it minimizes gate count/depth.
- (unexpectedly large depth = bad connectivity match).
- Splitting the process into two transpile calls (one keeping SWAPs visible, one fully native) separates **routing cost** (SWAPs needed for connectivity) from **gate-decomposition cost** (CX→CZ, H→RZ/SX), so you can attribute a bloated depth to one cause or the other.

## 3. SWAP Insertion

- If a CX is needed between non-adjacent qubits, the transpiler inserts SWAP gates to physically move states next to each other first.
- Each SWAP = 3 CX gates (or 3 native two-qubit gates after full decomposition) — expensive.

## 4. Gate Decomposition & Identities

- **H decomposition into native gates:** `H = RZ(π/2) · SX · RZ(π/2)`
- **CX ↔ CZ identity:** `CX(c,t) = (I ⊗ H_t) · CZ(c,t) · (I ⊗ H_t)`, i.e. sandwich the target with H before/after a CZ. Comes from the single-qubit identity `H Z H = X`.

- **Bridge gate identity** (CNOT bridging): implements a long-range CX using a middle qubit as a temporary relay, without permanently relocating it.
  ```
  CX(a,b)·CX(b,c)·CX(a,b)·CX(b,c) = CX(a,c)
  ```
  - Costs 4 CX total, and the middle qubit `b` ends up unchanged - cheaper than SWAP-there-and-back (7 CX), same cost as SWAP-and-leave-swapped (4 CX) but without needing to relabel qubits afterward.
 
## 5. Hadamard Gate (H)

$$H = \frac{1}{\sqrt{2}}\begin{pmatrix} 1 & 1 \\ 1 & -1 \end{pmatrix}$$

- $H|0\rangle = |+\rangle$, $H|1\rangle = |-\rangle$, and since $H^2=I$, also $H|+\rangle=|0\rangle$, $H|-\rangle=|1\rangle$ (two separate round trips, not a 4-cycle).
- **Geometrically:** a 180° rotation on the Bloch sphere about the axis exactly between the X-axis and Z-axis.

- Properties: unitary, Hermitian, self-inverse ($H^2=I$), $\det(H)=-1$, eigenvalues $\pm1$.
- Basis-change identities: $HXH=Z$, $HZH=X$, $HYH=-Y$.

## 6. Pauli Gates (X, Y, Z)

$$X=\begin{pmatrix}0&1\\1&0\end{pmatrix} \quad Y=\begin{pmatrix}0&-i\\i&0\end{pmatrix} \quad Z=\begin{pmatrix}1&0\\0&-1\end{pmatrix}$$

- Each is a 180° Bloch-sphere rotation about its own axis.
- **X (bit flip):** axis through |+⟩,|−⟩ (fixed); flips |0⟩↔|1⟩.
- **Z (phase flip):** axis through |0⟩,|1⟩ (fixed); flips |+⟩↔|−⟩.
- **Y (bit + phase flip):** axis perpendicular to both; $Y=iXZ$.


## 7. GHZ States

- **GHZ = Greenberger–Horne–Zeilinger** (physicists who first described the state, 1989 paper).
- $\frac{1}{\sqrt2}(|00\cdots0\rangle + |11\cdots1\rangle)$
- Measuring gives all-0s or all-1s only, each ~50%
  - **Fan-out:** branch outward from a center qubit in both directions (fewer layers if connectivity allows, e.g. 11 layers for a specific asymmetric fan-out).
  - **Chain (nearest-neighbor):** CX + SWAP down a line SWAPs happen to act as no-ops here since the swapped qubits always hold equal values.

## 8. GHZ Fidelity (measured, approximate)

```python
def ghz_fidelity(counts, n):
    total = sum(counts.values())
    correct = counts.get("0" * n, 0) + counts.get("1" * n, 0)
    return correct / total
```

- This function is a cheap proxy: fraction of shots landing on the two allowed GHZ outcomes.
- Catches bit-flip/decoherence noise (a shot like `00100` = qubit 2 decohered).

## 9. Heavy-Hex Lattice & Junctions

- IBM QPUs (Eagle, Heron, etc.) use a **heavy-hex lattice**: a hexagonal grid with an extra qubit inserted on every edge.
- Most qubits have degree 2 (sit on a line between two neighbors). Only qubits at an actual hexagon vertex get a 3rd connection (degree 3).
- Keeps max degree at 3 to reduce couplers, frequency-collision risk, and crosstalk, while keeping enough connectivity for stabilizer error correction.
- Degree-3 junctions appear at a repeating pattern reflecting the row structure (long horizontal chains + periodic vertical connectors linking rows).

## 10. Fake Backends

- `FakeTorino` (and similar `Fake*` classes) = local, offline snapshots of a real device's properties (coupling map, native gates, error rates, T1/T2).
- Runs via `AerSimulator` with the snapshot's noise model — realistic noisy results without using real QPU time.
- Same transpilation behavior as targeting the real backend (same routing, same decompositions).
- Calibration data is frozen at snapshot time; refreshing to live data may require appropriate account access for non-open backends.

## 11. What Actually Uses QPU Time

- **Does NOT use QPU time:** `Statevector(qc)`, `Operator`, `AerSimulator()`, any `Fake*` backend — all local/simulated.
- **DOES use QPU time:** submitting a job via `Sampler`/`Estimator` pointed at a real backend object from `QiskitRuntimeService`, e.g. `sampler.run([isa_circuit], shots=1024)` against `ibm_sherbrooke` etc.
- **Check usage:**
  - Website: Usage tab on the IBM Quantum Platform account dashboard (tracks rolling monthly QPU-seconds vs. plan cap).
  - Per-job: `job.usage()` after retrieving via `service.job("JOB_ID")`.

  https://quantum.cloud.ibm.com/instances