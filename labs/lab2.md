# Lab 2 Notes


## Transpilation / depth counting

- Splitting transpile into two calls (one keeping SWAPs visible with `basis_gates=["h","cx","swap"]`, one fully native) lets you separate **routing cost** (SWAPs = topology mismatch) from **decomposition cost** (H→RZ/SX, CX→CZ). Doesn't change what actually runs - only the native one gets executed on hardware. The SWAP-visible one is just so I can read off `count_ops()['swap']` before it all gets buried inside the native circuit's CZ count.
- `qc.depth()` = longest sequential chain of layers (gates sharing no qubits can share a layer).
- `qc.depth(lambda instr: len(instr.qubits) >= 2)` = depth counting only 2-qubit gates - matters more than total depth since 2Q gates are way noisier.
- `qc.num_nonlocal_gates()` = count of all multi-qubit gates in one line. Can also filter `circuit.data` manually with `instr.operation.num_qubits >= 2` if I want a breakdown by gate name via `Counter`.
- Watch out: `num_qubits` and `num_ancillas` are **properties** not methods - no parentheses, or it throws a TypeError.
- `num_ancillas` only counts qubits explicitly built via `AncillaRegister` - if I just used a normal `QuantumRegister` for my throwaway qubits it'll report 0 even though they're functionally ancillas.

## Measurement / basis stuff

- Measuring qubits in the same basis with nothing in between = order doesn't matter, they commute. Order only actually matters for mid-circuit measurement + classical feedback (dynamic circuits), or measuring the *same* qubit in two different non-commuting bases.
- Qiskit bitstrings are little-endian - rightmost char = qubit 0. Bit me on anything asymmetric.
- Measuring "in the X basis" = apply H right before measuring, since H swaps the X and Z axes. Turns "is this |+⟩ or |−⟩" into an ordinary Z-basis question.
- `repeated_x_meas_x_circuit` (H, 2n X's, H, measure): ideal result is always 0 since H puts it in |+⟩ which is an eigenstate of X, so the X gates literally do nothing to it. Any drift as n grows = phase/dephasing error specifically (not bit-flip), since bit-flip error is exactly what this circuit is immune to by construction.
- `QuantumCircuit(1,1)` = 1 qubit + 1 **separate** classical register, not "1 qubit with a label." Classical bit only gets written to via `measure()` - it's the bridge from quantum info to actual readable info.

## Ancilla

Ancilla = helper/throwaway qubit. Used temporarily, its own final state doesn't matter - only its effect on other qubits matters. Once it's done its job (e.g. the CX it participated in), safe to measure + reset it. This is literally what the dynamic circuit paper's disposable qubits are.

## Bridge gate identity

`CX(a,b)·CX(b,c)·CX(a,b)·CX(b,c) = CX(a,c)`, with qubit b ending up unchanged. Proved it by tracking plain bits through each step (works because CX has no phases, just permutes basis states, so checking on classical bit combos = checking the whole thing). Cheaper than full SWAP-there-and-back (4 CX vs 7) and doesn't require relabeling qubits after.

## Dynamic circuits GHZ paper (the big one)

- **Bell pair identity**: CX applied twice to the same Bell pair cancels back to an uncorrelated |+⟩|0⟩ (same self-inverse CX logic as the bridge identity). This is why pushing a second CX onto an already-entangled pair basically "un-does" it and reveals the pieces were separable all along.
- **Ancilla reuse (step iii)**: qubits that end up idle for a while (because their only CX got pushed later) can be repurposed, measured, and reset back to |0| so the same physical qubit gets recycled for the next block instead of needing a dedicated qubit per position.
- **The "cup" notation**: curvy wire = shorthand for a Bell pair, drawn as one continuous bent line instead of two straight wires + explicit H/CX. Gates can be "slid" along it visually because of a real underlying symmetry (see stabilizers below).
- **Why gates can slide across the cup**: NOT a general rule for any gate - specifically works for X and Z because |Φ+⟩ is a stabilizer state for X⊗X and Z⊗Z (applying X to both qubits, or Z to both, leaves it unchanged). That means X_a|Φ+⟩ = X_b|Φ+⟩, so which qubit the gate physically sits on doesn't matter. H does NOT get this - doesn't commute with the stabilizers, so it can't be slid.
- **Deferred measurement**: measuring the control qubit now and classically applying X vs. keeping it coherent and doing a real CX - genuinely equivalent, as long as nothing downstream ever recombines the two branches (like another H) afterward. The two branches sit on orthogonal states so they can never interfere again once nothing brings them back together.
- **H placement across blocks**: not a "swap" - each block is really its own fresh little Bell-pair prep. H just goes on whichever qubit is actually blank/fresh at that point (freshly reset ancilla), not on whichever one is drawn on top. The role of "which qubit is fresh" moves around each block because of the reset cycle, so H's position moves with it.
- **Steps vii → viii**: not new logic, just merging - instead of drawing each measurement→correction as its own separate sequential step, XOR all the relevant measurement outcomes together in one shot since classical computation is basically instant compared to quantum gates. Drawn as one vertical line instead of a chain. (viii) also just un-abbreviates the cups back into explicit H+CX, purely cosmetic.
- **4 classical registers** for a 4-qubit example = one clbit per physical qubit, not per measurement event. Each qubit's clbit gets overwritten if it's measured more than once (mid-circuit correction, then final readout) - fine because by the time the final measurement happens the earlier value already did its job.
