# Lab0 Notes

### SparsePauliOp (Qiskit)

- Represents a quantum operator/observable as a **sum of weighted Pauli strings** (tensor products of I, X, Y, Z), instead of a full dense matrix.
- Example: `"ZIX"` = Z⊗I⊗X on 3 qubits.
- **Why sparse:** dense matrices scale as 2ⁿ×2ⁿ, but most real observables only need a few Pauli terms -> efficient storage.
- Stored internally as: `.paulis` (PauliList) + `.coeffs` (complex Numpy array).

**Construction:**
```python
from qiskit.quantum_info import SparsePauliOp

op = SparsePauliOp(["ZZ", "XI"], coeffs=[1.0, 0.5])  # 1.0*ZZ + 0.5*XI
```

**Use case:** pass into `Estimator` to compute expectation values (e.g. `SparsePauliOp("Z")` -> ⟨Z⟩ of a circuit's output state).


### Transpilation 

The process of rewriting an abstract quantum circuit so it can actually run on a specific piece of quantum hardware, adapting it to that device's constraints while preserving the same measurement outcomes.

