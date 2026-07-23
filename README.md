# QSKIT 2026 — Course Notes

## Table of Contents
1. [Why Quantum? What Problem Are We Solving?](#why-quantum-what-problem-are-we-solving)
2. [Quantum Mechanics for Computation](#quantum-mechanics-for-computation)
3. [Qiskit Notebook Notes](#qiskit-notebook-notes)

---

## Why Quantum? What Problem Are We Solving?

### Key Quantum Concepts

| Concept | Meaning |
|---|---|
| **Superposition** | A quantum system existing in a complex linear combination of 2 states until it is measured |
| **Entanglement** | Information shared jointly between entangled pairs or groups |
| **Interference** | The combining of amplitudes so that possibilities can reinforce or cancel one another |

### Query Gate (Oracle)

The query gate (oracle) takes n bits input and m bits output:

$$U_f|y\rangle|x\rangle = |y \oplus f(x)\rangle|x\rangle$$

- **Input**: 2 registers of input, read right to left
- **Diagram**: ![query gate](images/query-gate.png)

> **Note:** quantum gates must be reversible.

### A Misconception

It is possible to feed in a superposition of all possible x values at once via a Hadamard gate:

$$\frac{1}{\sqrt{2^n}} \sum_x |x\rangle$$

Since quantum operations are linear, applying the circuit to this superposition of x gives a superposition of all corresponding output pairs simultaneously (**quantum parallelism**).

**HOWEVER** — quantum states are not directly readable. You must **measure** to get any information out. This causes the superposition to **collapse**, giving one random term via a probability (amplitude squared) — uniform in this case.

### Effect of the Hadamard Gate $H$

$$H|0\rangle = \frac{1}{\sqrt{2}}(|0\rangle + |1\rangle) \equiv |+\rangle$$

$$H|1\rangle = \frac{1}{\sqrt{2}}(|0\rangle - |1\rangle) \equiv |-\rangle$$

### Phase Kickback

$$U_f|-\rangle|x\rangle = (-1)^{f(x)}|-\rangle|x\rangle$$

- If $f(x) = 0$: $U_f|-\rangle|x\rangle = |-\rangle|x\rangle$
- If $f(x) = 1$: $U_f|-\rangle|x\rangle = -|-\rangle|x\rangle$

### The Parity Problem / Deutsch's Problem

![Deutsch's problem](image.png)

![Deutsch's algorithm](image-1.png)

![Solution to the Parity problem](image-3.png)

BQP: bounded-error quantum polynomial time
can be solved with high probability in polynomial time

### Deutsch-Jozsa problem

![Deutsch-Josza problem](image-4.png)

### Some More Algorithms

- Simon's algorithm
- Shor's algorithm
- Quantum phase estimation
- Grover's search

🔗 [quantumalgorithmzoo.org](https://quantumalgorithmzoo.org/)

### Quantum Applications

**Simulating nature**
- Spin models
- Materials
- Chemistry
- High energy physics

**Maths and data**
- Factoring
- Abelian hidden subgroup
- Supervised learning (classification)
- Unsupervised learning (clustering)
- Time series
- Regression
- Linear algebra

**Search and optimisation**
- Combinatorial optimisation
- Black-box optimisation
- General mixed integer

Other key application areas:
- Hamiltonian simulation
- Optimization
- Machine learning
- Differential equations
- Quantum error correction

🔗 [nature.com/articles/s41586-023-06096-3](https://www.nature.com/articles/s41586-023-06096-3)

### Quantum Error Correction

Encoding logical qubits in error-correcting codes allows one to detect and correct errors **during** circuit execution, at the cost of an overhead in the number of qubits.

**Fault-tolerant quantum computing** implements error-correcting codes in such a way that both the error correction and a universal set of logical operations can be carried out while reducing errors.

---

## Quantum Mechanics for Computation

### Dirac Notation

- A notation for representing vectors and matrices, widely used in quantum mechanics.
- Not exclusive to quantum information — it can represent classical information too.

**Kets — column vectors**

$$
|0\rangle = \begin{pmatrix} 1 \\ 0 \end{pmatrix}
\quad \text{and} \quad
|1\rangle = \begin{pmatrix} 0 \\ 1 \end{pmatrix}
$$

**Bras — row vectors**

$$
\langle 0| = \begin{pmatrix} 1 & 0 \end{pmatrix}
\quad \text{and} \quad
\langle 1| = \begin{pmatrix} 0 & 1 \end{pmatrix}
$$

> **Takeaway:** kets are columns, bras are their row-vector counterparts — the notation used to express and manipulate quantum (and classical) states.

### Measuring a System

- **Measuring** a system means observing it and identifying which classical state it's actually in.
- Once you observe a probabilistic state, you gain information — and use it to update the probabilities assigned to each classical state.

**Example: Coin Flip**

Before looking, an unflipped/unobserved coin is represented as a probabilistic mix of both outcomes:

$$
\begin{pmatrix} \frac{1}{2} \\ \frac{1}{2} \end{pmatrix} = \frac{1}{2}|\text{heads}\rangle + \frac{1}{2}|\text{tails}\rangle
$$

Once you look and see **tails**, the state collapses to a definite outcome:

$$
|\text{tails}\rangle = \begin{pmatrix} 0 \\ 1 \end{pmatrix}
$$

> **Takeaway:** measurement turns a probability distribution over possible states into certainty about one specific state.

### Deterministic Operations on Classical States

A deterministic operation transforms a classical state $a$ into $f(a)$, for some function $f: \Sigma \to \Sigma$.

**The four possible functions when Σ = {0, 1}:**

| $a$ | $f_1(a)$ | $f_2(a)$ | $f_3(a)$ | $f_4(a)$ |
|---|---|---|---|---|
| 0 | 0 | 0 | 1 | 1 |
| 1 | 0 | 1 | 0 | 1 |

- $f_1$: constant 0
- $f_2$: identity
- $f_3$: NOT (flips the bit)
- $f_4$: constant 1

### Representing Functions as Matrices

Any function can be written as a matrix $M$ acting on kets:

$$M|a\rangle = |f(a)\rangle$$

$$
M_1 = \begin{pmatrix} 1 & 1 \\ 0 & 0 \end{pmatrix}, \quad
M_2 = \begin{pmatrix} 1 & 0 \\ 0 & 1 \end{pmatrix}, \quad
M_3 = \begin{pmatrix} 0 & 1 \\ 1 & 0 \end{pmatrix}, \quad
M_4 = \begin{pmatrix} 0 & 0 \\ 1 & 1 \end{pmatrix}
$$

More generally, any such matrix can be constructed as:

$$M = \sum_{a \in \Sigma} |f(a)\rangle \langle a|$$

> **Takeaway:** every classical deterministic function on bits has a matrix representation, built from outer products of kets and bras, that acts on states exactly the same way the function acts on classical values.

### Stochastic (Probabilistic) Operations

- A classical operation $M$ doesn't need to be deterministic — it only needs to map probability vectors to probability vectors.
- Such operations are described by **stochastic matrices**.

**Stochastic matrix — definition:**

A square matrix is *stochastic* if:
- All entries are nonnegative, **and**
- Each column sums to 1 (i.e., each column is itself a valid probability vector).

**Interpretation:** a probabilistic operation can be understood as a **random choice among deterministic operations** — a weighted mixture of several deterministic matrices.

**Example:**

$$
\begin{pmatrix} 1 & \frac{1}{2} \\ 0 & \frac{1}{2} \end{pmatrix}
=
\frac{1}{2}\begin{pmatrix} 1 & 0 \\ 0 & 1 \end{pmatrix}
+
\frac{1}{2}\begin{pmatrix} 1 & 1 \\ 0 & 0 \end{pmatrix}
$$

Here, the stochastic matrix is a 50/50 mix of the identity operation and the "constant 0" operation from the earlier example.

> **Takeaway:** stochastic matrices generalize deterministic operations by allowing randomness — each column represents the probability distribution over outcomes for a given input state.

### Combining Two Systems

If there are two systems:
- $X$ with state set $\Sigma$
- $Y$ with state set $\Gamma$

then the state set of the combined system $(X, Y)$ is the **Cartesian product**:

$$\Sigma \times \Gamma = \{(a, b) : a \in \Sigma \text{ and } b \in \Gamma\}$$

**Example: two bits.** If $X$ and $Y$ are both bits, then $\Sigma = \Gamma = \{0, 1\}$, so:

$$\Sigma \times \Gamma = \{(0,0), (0,1), (1,0), (1,1)\}$$

These are usually written as bitstrings:

$$\Sigma \times \Gamma = \{00, 01, 10, 11\}$$

> **Takeaway:** the combined state space of two systems is just every possible pairing of their individual states — for two bits, this gives the familiar four 2-bit strings.

### The Tensor Product

In the language of vectors and matrices, multiple systems are combined using the **tensor product**.

**Definition.** Given two vectors:

$$
|\phi\rangle = \sum_{a \in \Sigma} \alpha_a |a\rangle
\quad \text{and} \quad
|\psi\rangle = \sum_{b \in \Gamma} \beta_b |b\rangle
$$

the tensor product $|\phi\rangle \otimes |\psi\rangle$ is defined as:

$$|\phi\rangle \otimes |\psi\rangle = \sum_{(a,b) \in \Sigma \times \Gamma} \alpha_a \beta_b |ab\rangle$$

> **Takeaway:** the tensor product combines the amplitudes of two independent states by multiplying every pair of coefficients ($\alpha_a \beta_b$) and assigning each product to the corresponding joint basis state $|ab\rangle$ — this is how you build the state of a combined system from its parts.

- It's common to drop the $\otimes$ symbol: $|\phi\rangle|\psi\rangle$ is shorthand for $|\phi\rangle \otimes |\psi\rangle$.
- The tensor product is **bilinear** — linear in each argument separately.

**Linearity in the first argument:**

$$(|\phi_1\rangle + |\phi_2\rangle)|\psi\rangle = |\phi_1\rangle|\psi\rangle + |\phi_2\rangle|\psi\rangle$$

$$(\alpha|\phi\rangle)|\psi\rangle = \alpha(|\phi\rangle|\psi\rangle)$$

**Linearity in the second argument:**

$$|\phi\rangle(|\psi_1\rangle + |\psi_2\rangle) = |\phi\rangle|\psi_1\rangle + |\phi\rangle|\psi_2\rangle$$

$$|\phi\rangle(\alpha|\psi\rangle) = \alpha(|\phi\rangle|\psi\rangle)$$

> **Takeaway:** the tensor product respects addition and scalar multiplication in both slots — you can distribute sums and pull out scalars just like with ordinary multiplication, which makes it straightforward to expand combined states term by term.

### Exponential State Space of N Bits

The state space of $N$ bits is the set of all bitstrings of length $N$, which has size $2^N$ — **exponential** in $N$.

**Example: 10 bits**

$$0000000000, \; 0000000001, \; 0000000010, \; 0000000011, \; \dots, \; 1111111110, \; 1111111111$$

($2^{10} = 1024$ total bitstrings)

**Why this matters:** the probability vector for a system of **500 bits** has length $2^{500}$ — a number larger than the estimated count of atoms in the observable universe. Because of this, computer programs never explicitly store all the probabilities for large systems.

> **Takeaway:** state spaces grow exponentially with the number of bits, which is why classical simulation of large systems (and, by extension, quantum systems) quickly becomes computationally infeasible — probabilities have to be represented implicitly rather than stored outright.

### Sampling Without Storing the Full Distribution

Rather than explicitly storing a probability vector with $2^{500}$ entries, a simple Python script can sample a bitstring directly from the (uniform) distribution:

```python
import random

n_bits = 500
bits = random.getrandbits(n_bits)
print(format(bits, f"0{n_bits}b"))
```

This generates a random 500-bit string, effectively sampling from the distribution:

$$
\begin{pmatrix} \frac{1}{2^{500}} \\ \frac{1}{2^{500}} \\ \vdots \\ \frac{1}{2^{500}} \end{pmatrix}
\Bigg\} \, 2^{500} \text{ entries}
$$

> **Takeaway:** even though the full probability vector is astronomically large and impossible to store, sampling from a uniform distribution over bitstrings just requires generating random bits directly — no need to ever materialize the vector itself.

---

## Qiskit Notebook Notes

### 1. Single-Qubit Rotation with `RXGate`

```python
from qiskit.circuit.library import XGate, RXGate

statevec = Statevector.from_label("0")
statevec = statevec.evolve(RXGate(0.4))
statevec.sample_counts(1000)
```

- **`XGate`** — fixed Pauli-X (bit-flip) gate, equivalent to `RXGate(π)` up to global phase.
- **`RXGate(theta)`** — parameterized rotation about the X-axis by `theta` radians.
- Starting from `|0⟩` and applying `RX(0.4)` tilts the state slightly off the north pole on the Bloch sphere, producing a superposition weighted mostly toward `|0⟩`.
- `sample_counts(1000)` draws 1000 simulated measurement shots.

**Result:** `{'0': 957, '1': 43}` → ~95.7% / 4.3%.

**Theory check:** for `RX(θ)` from `|0⟩`, P(`|1⟩`) = sin²(θ/2).
For θ = 0.4: sin²(0.2) ≈ 3.95% — close to the observed 4.3%, well within statistical noise for n = 1000.

### 2. Extracting Bloch Sphere Angles from a Statevector

```python
import cmath
import math

def bloch_angles(a: complex, b: complex):
    global_phase = cmath.rect(1, cmath.phase(a))
    a *= global_phase.conjugate()
    b *= global_phase.conjugate()
    assert math.isclose(a.imag, 0, abs_tol=...)
    return 2 * math.acos(a.real), cmath.phase(b)
```

Any single-qubit pure state can be written (up to global phase) as:

$$|\psi\rangle = \cos(\theta/2)|0\rangle + e^{i\phi}\sin(\theta/2)|1\rangle$$

- `theta` = polar angle (tilt from `|0⟩` at the north pole)
- `phi` = azimuthal angle (rotation around the equator)

**How the function works:**
1. **Remove global phase** — multiply `a` and `b` by the conjugate of `a`'s phase factor, so `a` becomes real and non-negative (global phase has no physical meaning).
2. **Sanity check** — confirm `a` is now purely real.
3. **Recover angles** — since `a.real = cos(θ/2)`, `θ = 2·acos(a.real)`; since global phase is removed, `φ = phase(b)` directly.

**Applying it to a circuit:**

```python
from qiskit.visualization import plot_bloch_vector

statevec = Statevector.from_label("0")
statevec = statevec.evolve(RXGate(np.pi/2))
statevec = statevec.evolve(RZGate(np.pi/2))
statevec = statevec.evolve(RYGate(np.pi/4))
theta, phi = bloch_angles(*statevec)

plot_bloch_vector([1, theta, phi], coord_type="spherical")
```

- Starts at `|0⟩`, then applies `RX(π/2)` → `RZ(π/2)` → `RY(π/4)` in sequence, each rotation tilting/spinning the state on the Bloch sphere.
- `bloch_angles(*statevec)` unpacks the two amplitudes and converts them to `(theta, phi)`.
- `plot_bloch_vector([1, theta, phi], coord_type="spherical")` renders a unit-length vector (pure state → on the sphere's surface) at that polar/azimuthal position.

