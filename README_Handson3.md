# Hands-on 3: Quantum Mechanics Postulates, Dirac Notation & the Bloch Sphere

[![Python](https://img.shields.io/badge/Python-3.9%2B-blue?logo=python&logoColor=white)](https://www.python.org/)
[![Jupyter](https://img.shields.io/badge/Jupyter-Notebook-orange?logo=jupyter&logoColor=white)](https://jupyter.org/)
[![NumPy](https://img.shields.io/badge/NumPy-2.0%2B-013243?logo=numpy&logoColor=white)](https://numpy.org/)
[![Matplotlib](https://img.shields.io/badge/Matplotlib-3.x-11557c?logo=matplotlib&logoColor=white)](https://matplotlib.org/)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](../../LICENSE)
[![Series](https://img.shields.io/badge/Series-Quantum_Computing_Education-6929C4?style=flat-square)](https://github.com/Trinexis-Quantum/Quantum-Concepts)

---

## Overview

Quantum mechanics is often taught by first absorbing a wall of axioms and then, much later, seeing them in action. This notebook reverses that order. By the time you open it, you will have already *watched* quantum behaviour in the two landmark experiments of the course, interference patterns in the double-slit experiment and the puzzling, repeatable outcomes of cascaded Stern-Gerlach (SG) magnets. The purpose of this notebook is to show you that those real experimental observations are not mysterious accidents: they are precise, quantitative consequences of five well-defined mathematical rules called the **postulates of quantum mechanics**.

Think of the postulates as the grammar of a new language. The SG experiments taught you words and phrases, "spin up", "incompatible measurements", "erasure restores interference". The postulates give you the grammar that explains why those phrases make sense and lets you extend them to new situations you have never seen before. Once you have the grammar, you have a superpower: you can reason about *any* quantum system, including quantum computers, using the same small set of ideas.

The tools the notebook uses are deliberately minimal, only NumPy (for linear algebra) and Matplotlib (for plots). No quantum-computing framework is needed. Every quantum object, a qubit state, a measurement, a gate, the Bloch sphere, is built from scratch in fewer than twenty lines of Python. By doing this yourself, you see that quantum mechanics is not magic wrapped in a library: it is linear algebra, dressed in elegant notation, pointing at deep physics.

---

## Learning Objectives

By the end of this notebook you will be able to:

- **State and explain** each of the five postulates of quantum mechanics in your own words, and identify which part of the Stern-Gerlach experiment each postulate encodes.
- **Write and normalise** a qubit state in Dirac (bra-ket) notation and verify it satisfies the unit-norm requirement.
- **Explain the global-phase ambiguity**: why $|\psi\rangle$ and $e^{i\theta}|\psi\rangle$ are the same physical state, and verify it numerically.
- **Compute Born-rule probabilities** for any observable on any state, simulate single-shot measurements, and reproduce the 50/50 SG-z outcome for $|{+}\rangle$.
- **Demonstrate collapse** in code: show repeatability after measurement, and show that a second incompatible measurement randomises the outcome again.
- **Build the inner product and outer product** in NumPy; verify conjugate symmetry, orthonormality of the computational and $x$-bases, and the completeness relation $\sum_i |i\rangle\langle i| = I$.
- **Check whether an operator is Hermitian or unitary** numerically, compute eigenvalues and eigenvectors, and evaluate expectation values.
- **Decompose any single-qubit operator** in the Pauli basis $\{I, \sigma_x, \sigma_y, \sigma_z\}$ using the trace formula.
- **Compute Bloch vectors** as triples of Pauli expectation values, and visualise any set of qubit states on the Bloch sphere.
- **Interpret a quantum gate as a rotation** on the Bloch sphere, and construct continuous-rotation unitaries via the matrix exponential $e^{-i\theta(\hat{n}\cdot\vec{\sigma})/2}$.

---

## Background and Theory

### The Problem the Postulates Solve

Classical physics deals with objects that have definite properties at all times, whether or not we look at them. An electron in a classical world would have a definite spin direction. The SG experiments prove this picture is wrong: an electron prepared in the "$+x$" state has a completely random $z$-outcome, not because we are ignorant of a hidden spin direction, but because there *is* no definite $z$-spin to be ignorant of. The erasure experiment makes this vivid: if the electron "secretly" had a definite $z$-value, a later quantum eraser could never restore the $x$-interference pattern, but it does.

The five postulates are the minimal set of rules that correctly describe this behaviour for every experiment ever performed.

---

### Postulate 1, State Space: The Qubit Lives in $\mathbb{C}^2$

A **quantum state** is not a list of property values, it is a *unit vector* in a complex inner-product space (a Hilbert space). For a single qubit (spin-1/2 particle, polarised photon, two-level atom, the physics does not matter) that space is two-dimensional complex space $\mathbb{C}^2$:

$$|\psi\rangle = \alpha|0\rangle + \beta|1\rangle, \qquad |\alpha|^2 + |\beta|^2 = 1.$$

The entries $\alpha, \beta \in \mathbb{C}$ are called **probability amplitudes**. The constraint that their squared magnitudes sum to one is not optional book-keeping, it is the mathematical expression of the physical fact that *something* must happen when you measure.

**Analogy.** Imagine a qubit as an arrow of fixed unit length planted somewhere on a sphere. Classically you would say "the arrow points north (0) or south (1)." The qubit's arrow can point *anywhere* on the sphere, that is superposition. The normalisation constraint just says the arrow always has length one.

**Global phase.** Notice that $e^{i\theta}|\psi\rangle$ has exactly the same $|\alpha|^2$ and $|\beta|^2$ as $|\psi\rangle$, for any real angle $\theta$. Every observable quantity depends only on these squared magnitudes, so no experiment can distinguish $|\psi\rangle$ from $e^{i\theta}|\psi\rangle$. They are the *same physical state*. This is the global-phase ambiguity.

---

### Postulate 2, Observables are Hermitian Operators

Every physical quantity you can measure, spin along $z$, energy, momentum, is represented by a **Hermitian operator** $\hat{A} = \hat{A}^\dagger$. The possible numerical outcomes of that measurement are the *eigenvalues* of $\hat{A}$, and the states in which the outcome is guaranteed (no randomness) are its *eigenvectors*.

For a qubit, "spin along $z$" is the Pauli $\sigma_z$ matrix; "spin along $x$" is $\sigma_x$. Both have eigenvalues $\pm 1$ (the two SG beam ports), but their eigenvectors are different, that is why the two magnets are *incompatible*.

**Why Hermitian?** Two key theorems follow from the Hermitian condition automatically:

1. **Real eigenvalues**, measurement outcomes are real numbers, not complex.
2. **Orthogonal eigenvectors**, the SG beams never overlap. If you are definitely in the "$+1$" eigenstate, you have zero probability of the "$-1$" outcome.

---

### Postulate 3, The Born Rule: Squaring Amplitudes Gives Probabilities

Given state $|\psi\rangle$ and observable $\hat{A}$ with spectral decomposition $\hat{A} = \sum_i \lambda_i |\lambda_i\rangle\langle\lambda_i|$, the probability of obtaining outcome $\lambda_i$ is:

$$P(\lambda_i) = |\langle \lambda_i | \psi \rangle|^2.$$

The complex number $\langle \lambda_i | \psi \rangle$ is the **amplitude**. Crucially, amplitudes can interfere with each other *before* being squared. This is the entire mechanism behind the double-slit fringes: the two path amplitudes (upper slit and lower slit) add as complex numbers, and only the squared magnitude of the sum is the observed intensity. Classical probabilities obey $P(A \text{ or } B) = P(A) + P(B)$; amplitudes obey $A(A \text{ or } B) = A(A) + A(B)$, and then $P = |A|^2$. The cross term that appears when you square the sum is interference.

**The normalisation connection.** Because eigenvectors of $\hat{A}$ form a complete orthonormal set:

$$\sum_i P(\lambda_i) = \sum_i |\langle\lambda_i|\psi\rangle|^2 = \langle\psi|\psi\rangle = 1.$$

Probabilities summing to one is the same statement as the state being normalised.

---

### Postulate 4, Collapse: Measurement Updates the State

After obtaining outcome $\lambda_i$, the state is no longer $|\psi\rangle$. It **collapses** to the corresponding eigenvector $|\lambda_i\rangle$. This encodes **repeatability**: repeat the same measurement immediately and you get the same result with certainty. In the SG experiment, an atom that exits the "$+z$" port of the first magnet always exits the "$+z$" port of a second magnet, it has been "prepared" in a definite state.

Collapse is **basis-specific**: collapsing to a $z$-eigenstate makes the $x$-value completely random again (it puts the atom into a superposition in the $x$-basis). This is exactly why inserting a $z$-measurement between two $x$-magnets destroys the $x$-correlation seen without the middle magnet.

Mathematically, the post-measurement state can be written using the projector $\hat{P}_i = |\lambda_i\rangle\langle\lambda_i|$:

$$|\psi_{\text{after}}\rangle = \frac{\hat{P}_i|\psi\rangle}{\|\hat{P}_i|\psi\rangle\|}.$$

---

### Postulate 5, Time Evolution is Unitary

Between measurements, a quantum state evolves as:

$$|\psi(t)\rangle = \hat{U}(t)|\psi(0)\rangle, \qquad \hat{U}^\dagger \hat{U} = I.$$

The **unitary** condition $\hat{U}^\dagger\hat{U} = I$ enforces two things simultaneously: (a) total probability is conserved (the evolved state stays normalised), and (b) evolution is reversible (every quantum gate can be undone). In quantum computing, every gate, Hadamard, CNOT, Toffoli, is a unitary operator.

**The bridge to physics.** The unitary arises from the Schrodinger equation: for a time-independent Hamiltonian $\hat{H}$ (itself Hermitian), the solution is $\hat{U}(t) = e^{-i\hat{H}t/\hbar}$. The matrix exponential of any Hermitian operator is always unitary. This is implemented in the notebook using eigendecomposition:

$$e^{-i\hat{H}t} = V \, \mathrm{diag}(e^{-i\lambda_k t}) \, V^\dagger,$$

where $\hat{H} = V \, \mathrm{diag}(\lambda_k) \, V^\dagger$.

---

### Dirac Notation

Paul Dirac invented a compact notation that makes quantum linear algebra feel almost like arithmetic:

| Symbol | Name | NumPy equivalent | Meaning |
|--------|------|-------------------|---------|
| $\|\psi\rangle$ | ket | 1-D complex array (column) | state vector |
| $\langle\psi\|$ | bra | conjugate of the array (row) | dual vector |
| $\langle\phi\|\psi\rangle$ | bracket | `np.vdot(phi, psi)` | amplitude / overlap (a number) |
| $\|\psi\rangle\langle\phi\|$ | outer product | `np.outer(psi, phi.conj())` | operator (a matrix) |
| $\hat{A}\|\psi\rangle$ | action | `A @ psi` | matrix-vector product |

Note: `np.vdot` conjugates its *first* argument, matching the bra on the left.

---

### Pauli Matrices and the Bloch Sphere

The three **Pauli matrices**:

$$\sigma_x = \begin{pmatrix}0&1\\1&0\end{pmatrix}, \quad \sigma_y = \begin{pmatrix}0&-i\\i&0\end{pmatrix}, \quad \sigma_z = \begin{pmatrix}1&0\\0&-1\end{pmatrix}$$

together with the identity $I$ span the entire space of $2\times2$ matrices. Every single-qubit operator can be written as $M = c_I I + c_x\sigma_x + c_y\sigma_y + c_z\sigma_z$, with coefficients extracted by the trace formula $c_k = \frac{1}{2}\mathrm{Tr}(\sigma_k M)$.

The **Bloch sphere** turns this decomposition geometric. Every pure qubit state (up to global phase) can be written:

$$|\psi\rangle = \cos\tfrac{\theta}{2}|0\rangle + e^{i\varphi}\sin\tfrac{\theta}{2}|1\rangle,$$

and mapped to a point on the unit sphere with Cartesian coordinates:

$$\vec{r} = \bigl(\langle\sigma_x\rangle,\; \langle\sigma_y\rangle,\; \langle\sigma_z\rangle\bigr) = (\sin\theta\cos\varphi,\; \sin\theta\sin\varphi,\; \cos\theta).$$

The north pole is $|0\rangle$, the south pole is $|1\rangle$, and $|{+}\rangle$ sits on the equator. A quantum gate is literally a rotation of this sphere; the Hadamard sends the north pole to the equator along the $x$-axis.

---

## Prerequisites

**Mathematics:**
- Complex numbers: magnitude, phase, conjugation.
- Linear algebra: vectors, matrices, matrix multiplication, eigenvalues and eigenvectors.
- Basic calculus (for the Schrodinger equation narrative; not required for the code).

**Programming:**
- Python 3.9 or later.
- NumPy basics: array creation, `@` for matrix multiplication, `np.linalg.eigh`.
- Matplotlib basics: `plt.bar`, `plt.figure`, 3D plotting is handled inside the provided helper functions.

**Course context:**
- You should have attended (or reviewed) the lectures covering the double-slit experiment and the cascaded Stern-Gerlach experiments (Demo 1-2 in this series). Those demos are the motivation for every postulate here.

**No prior quantum mechanics is assumed.** If you have seen any of it before, this notebook will crystallise it; if you have not, this notebook is designed to be your first encounter.

---

## Notebook Walkthrough

### Section 1, Setup and Helper Tools

The notebook begins by importing only NumPy and Matplotlib and defining three small helper functions: `fmt_c` (compact complex-number formatting), `show_ket` (prints a state vector in both column and Dirac form), and `show_matrix` (prints a matrix row by row). These exist purely to reduce visual clutter so the physics code stays readable. The computational basis kets $|0\rangle$ and $|1\rangle$ and the $x$-basis kets $|{+}\rangle$ and $|{-}\rangle$ are built immediately, these four vectors will appear in almost every section.

**Why start here?** Having these basis states as named variables means you can focus on the physics of each postulate instead of re-deriving the vectors every time.

---

### Section 2, The Five Postulates

Each postulate follows the same pattern: state the rule in formal language, connect it to the SG experiment you already watched, then verify it in four to ten lines of Python.

**Postulate 1 (State space)** demonstrates normalisation and the global-phase ambiguity. It makes a point many textbooks gloss over: $|\psi\rangle$ and $e^{i\theta}|\psi\rangle$ look completely different as complex arrays but produce identical squared magnitudes. The misconception box drives home that $\frac{1}{\sqrt{2}}(|0\rangle+|1\rangle)$ is *not* classical ignorance, the erasure experiment proves it.

**Postulate 2 (Observables)** diagonalises $\sigma_z$ and $\sigma_x$ with `np.linalg.eigh` and prints the eigenvalues ($\pm1$, matching the two SG ports) and eigenvectors (the four cardinal states). The `eigh` call is deliberate: for Hermitian matrices it is guaranteed to return real eigenvalues and orthonormal eigenvectors, a numeric demonstration of the two Hermitian theorems.

**Postulate 3 (Born rule)** has two demonstrations. First, the analytic `born_probs` function computes $P(\lambda_i) = |\langle\lambda_i|\psi\rangle|^2$ for $|{+}\rangle$ measured along $z$ and confirms the 50/50 split. Second, a Monte-Carlo simulation runs 2000 single-shot measurements and histograms them, giving students a concrete feel for how probability emerges from many individual random events.

**Postulate 4 (Collapse)** measures $|{+}\rangle$ once to get a collapsed state, then repeats the same measurement ten times to show the result is locked in. Then it measures the *same collapsed state* along $x$ ten times to show the randomness returns, the basis-specificity of collapse coded directly.

**Postulate 5 (Unitarity)** builds the Hadamard gate, confirms $H^\dagger H = I$, and shows $H|0\rangle = |{+}\rangle$. The misconception box explains why "quantum parallelism" is not classical parallel branches, a conceptual trap many newcomers fall into.

---

### Section 3, Ket, Bra, Operators: Dirac Notation

This section is a translation table between the physics notation and the NumPy operations. It builds a ket, prints its bra (the conjugate), then computes all four matrix elements $A_{ij} = \langle i|\hat{A}|j\rangle$ one by one. The goal is to make the index calculation feel completely transparent before higher-level functions start hiding it.

---

### Section 4, Inner Products and Outer Products

**Inner product** ($\langle\phi|\psi\rangle$, a scalar): The notebook checks three key special values, $\langle 0|{+}\rangle = 1/\sqrt{2}$ (the $z$-to-$x$ overlap that gives the 50/50 Born probability), $\langle 0|1\rangle = 0$ (orthogonality), and $\langle{+}|{+}\rangle = 1$ (normalisation). It then verifies conjugate symmetry $\langle\phi|\psi\rangle = \overline{\langle\psi|\phi\rangle}$.

The Gram-matrix cell shows the full $2\times2$ matrix of pairwise inner products for both the computational basis and the $x$-basis. Both are the identity, both are orthonormal bases for the same space $\mathbb{C}^2$, just tilted relative to each other.

**Outer product** ($|\psi\rangle\langle\phi|$, a matrix): The projector $P_0 = |0\rangle\langle 0|$ is built with `np.outer`, verified to be idempotent ($P_0^2 = P_0$), and the completeness relation $P_0 + P_1 = I$ is checked, then verified again in the $x$-basis. Finally, collapse is re-expressed as "project then renormalise", which unifies postulate 4 with the algebra.

---

### Section 5, Hermitian and Unitary Operators

This section adds depth to the guarantees stated in Postulate 2. A generic Hermitian matrix with complex off-diagonal entries is constructed and diagonalised; the eigenvalues are confirmed real and the eigenvectors orthonormal.

**Expectation value** $\langle\hat{A}\rangle = \langle\psi|\hat{A}|\psi\rangle$ is computed analytically and then cross-checked by averaging 5000 Monte-Carlo measurements. The $z$-expectation of $|{+}\rangle$ is exactly zero, the average of equally likely $+1$ and $-1$, as you would predict from the SG 50/50 split.

**Unitary geometry** shows that $H$ preserves inner products and norms: $\langle H\phi|H\psi\rangle = \langle\phi|\psi\rangle$. This is the mathematical reason a gate can act on a state without "leaking" probability.

**The Hermitian-Unitary bridge** $\hat{U} = e^{-i\hat{H}t}$ is implemented using eigendecomposition, making the Schrodinger equation concrete. The result is confirmed unitary.

---

### Section 6, Pauli Matrices and the Pauli Basis

All four Paulis (including identity) are displayed. A sweep confirms that each of $\sigma_x, \sigma_y, \sigma_z$ is simultaneously Hermitian, unitary, traceless, squares to the identity, and has eigenvalues $\pm1$, a rare combination that makes them special.

**Algebra** verifies the cyclic commutator $[\sigma_x,\sigma_y] = 2i\sigma_z$ and the anticommutator $\{\sigma_x,\sigma_y\} = 0$. The non-zero commutator is the algebraic fingerprint of incompatibility: observables that commute can be simultaneously sharp; those that anticommute cannot. The SG-x and SG-z incompatibility is encoded in $[\sigma_x,\sigma_z] \ne 0$.

**Pauli decomposition** uses the trace formula to express the Hadamard gate as $H = \frac{1}{\sqrt{2}}(\sigma_x + \sigma_z)$, a result that will recur in quantum algorithm analysis. The same function works on an arbitrary Hermitian matrix, giving real coefficients.

---

### Section 7, The Bloch Sphere

The `bloch_vector` function computes $(\langle\sigma_x\rangle, \langle\sigma_y\rangle, \langle\sigma_z\rangle)$ for any state. The six cardinal states are displayed: north pole ($|0\rangle$), south pole ($|1\rangle$), $\pm x$-axis ($|\pm\rangle$), and $\pm y$-axis ($|i\rangle$, $|{-i}\rangle$).

The `plot_bloch` function renders a wireframe sphere in 3D with colour-coded quiver arrows, each state becomes a visible arrow. Students can call this with any list of states and immediately see the geometry.

**Gates as rotations** shows the Hadamard moving the north-pole arrow to the equator. The `rotation` function builds $R_{\hat{n}}(\theta) = e^{-i\theta(\hat{n}\cdot\vec{\sigma})/2}$ and sweeps $|0\rangle$ from north pole to south pole through six frames of a $y$-axis rotation, printing the $z$-coordinate at each step (1 at the north, 0 at the equator, $-1$ at the south pole). A gate is literally a rotation on the sphere.

---

### Section 8, Exercises

Six exercises invite students to apply every tool introduced:

1. Build and normalise a new superposition state; compute Born probabilities.
2. Compare analytic expectation with Monte-Carlo average.
3. Verify incompatibility via commutators.
4. Decompose the $S$-gate in the Pauli basis; check Hermitian/unitary properties.
5. Visualise the $\sigma_y$ eigenstates on the Bloch sphere.
6. Construct the NOT gate as a rotation and compare with $\sigma_x$.

Suggested solutions are provided in a collapsed code block, students are encouraged to try before looking.

---

## Key Takeaways

- **Amplitudes are not probabilities.** They are complex numbers that can interfere. The Born rule squares them to get probabilities; interference is the cross term that disappears in that squaring only if you look at multiple paths in isolation.
- **Superposition is not ignorance.** $|{+}\rangle$ is not "secretly $|0\rangle$ or $|1\rangle$, we just don't know which." If it were, quantum erasure could not restore interference. The quantum state is the complete description, not a partial one.
- **Collapse is basis-specific.** Collapsing along $z$ randomises the $x$-outcome again. Definite in one basis means maximally uncertain in any mutually unbiased basis, this is the Heisenberg uncertainty principle in action for spin.
- **Hermitian and unitary operators play complementary roles.** Hermitian operators are the observables (measurement); unitary operators are the gates (evolution). They are linked by the matrix exponential $e^{-i\hat{H}t}$.
- **Every single-qubit operator is a Pauli combination.** The set $\{I, \sigma_x, \sigma_y, \sigma_z\}$ spans the full space of $2\times2$ matrices. Understanding the Paulis means understanding all single-qubit physics.
- **The Bloch sphere makes geometry visible.** The Bloch vector $\vec{r} = (\langle\sigma_x\rangle, \langle\sigma_y\rangle, \langle\sigma_z\rangle)$ converts abstract state algebra into arrows on a sphere, and gates into rotations, a geometric intuition that carries directly into circuit design.
- **"Quantum parallelism" is interference engineering, not classical branching.** Speed-up in quantum algorithms comes from constructing unitaries that amplify the amplitude of right answers and cancel the amplitude of wrong answers through destructive interference, not from running many classical computations simultaneously.

---

## Further Reading and Citations

1. **Nielsen, M. A., & Chuang, I. L.** (2010). *Quantum Computation and Quantum Information* (10th Anniversary Edition). Cambridge University Press., The canonical reference for all five postulates, Dirac notation, and the Bloch sphere (Chapters 1-2). Often abbreviated "NC" in the literature.

2. **Sakurai, J. J., & Napolitano, J.** (2020). *Modern Quantum Mechanics* (3rd ed.). Cambridge University Press., Covers bra-ket formalism and the Stern-Gerlach experiments in depth; Chapter 1 is an excellent companion to this notebook.

3. **Preskill, J.** (1998). *Lecture Notes on Quantum Computation*, Chapter 2: Foundations of Quantum Theory. California Institute of Technology. Available at: [http://theory.caltech.edu/~preskill/ph229/](http://theory.caltech.edu/~preskill/ph229/), Graduate-level but clearly written; the Bloch-sphere and density-matrix treatment is particularly thorough.

4. **Wilde, M. M.** (2017). *Quantum Information Theory* (2nd ed.). Cambridge University Press. arXiv preprint: [arXiv:1106.1445](https://arxiv.org/abs/1106.1445), Excellent for the mathematical underpinnings of Hilbert spaces, operators, and measurement theory (Chapters 3-4).

5. **Griffiths, D. J., & Schroeter, D. F.** (2018). *Introduction to Quantum Mechanics* (3rd ed.). Cambridge University Press., The most accessible introduction to wave-function postulates and spin-1/2 systems; Chapter 4 covers spin and the SG experiment with physical intuition.

6. **Dirac, P. A. M.** (1958). *The Principles of Quantum Mechanics* (4th ed.). Oxford University Press., The original source of bra-ket notation; worth reading Chapters 1-2 to appreciate the economy of the formalism.

7. **Feynman, R. P., Leighton, R. B., & Sands, M.** (1965). *The Feynman Lectures on Physics, Volume III: Quantum Mechanics*. Addison-Wesley. Available free at [https://www.feynmanlectures.caltech.edu/III_toc.html](https://www.feynmanlectures.caltech.edu/III_toc.html), Chapters 5-6 on spin-1/2 provide physical intuition for every postulate here; Feynman's treatment of two-state systems is unmatched for clarity.

---

<!-- 
SEO TAGS / KEYWORDS FOR DISCOVERABILITY:
quantum mechanics postulates, Dirac notation, bra-ket notation, qubit, Bloch sphere,
quantum computing tutorial, Born rule, wave function collapse, superposition, entanglement,
Hermitian operator, unitary operator, Pauli matrices, quantum measurement, Stern-Gerlach,
quantum state vector, inner product Hilbert space, quantum gates rotation, Hadamard gate,
Pauli basis decomposition, expectation value quantum, quantum probability amplitude,
NumPy quantum mechanics, Python quantum computing, quantum computing, jupyter quantum,
single qubit operations, quantum collapse postulate, completeness relation, projector operator,
Schrodinger equation unitary, matrix exponential quantum, qubit state normalization,
quantum superposition misconception, global phase qubit, quantum incompatibility commutator
-->

---

## Related Notebooks in This Series

| # | Notebook | Topic |
|---|----------|-------|
| 1-2 | [Demo1-2: Double Slit and Stern-Gerlach](../Demo1-2_Double_Slit_and_Stern_Gerlach.ipynb) | Wave-particle duality, interference, SG cascades, the experimental motivation for this notebook |
| **3** | **Demo3: QM Postulates, Bra-Ket, Bloch Sphere** *(this notebook)* | **The five postulates, Dirac notation, Pauli matrices, Bloch sphere** |
| 4 | [Demo4: Bloch Sphere and Density Matrix](../Demo4_BlochSphere_DensityMatrix.ipynb) | Mixed states, density operators, purity, decoherence |
| 5 | [Demo5: Purity, Coherence, Entanglement](../Demo5_Purity_Coherence_Entanglement.ipynb) | Two-qubit tensor products, Bell states, entanglement measures |
| 6 | [Demo6: Noise and Information Measures](../Demo6_Noise_and_Information_Measures.ipynb) | Quantum channels, von Neumann entropy, quantum mutual information |
| 7 | [Demo7: Quantum Gates](../Demo7_Quantum_Gates_Demo.ipynb) | Single- and multi-qubit gate zoo, circuit notation |
| 8 | [Demo8: Quantum Circuits, Entangling Gates, WHT](../Demo8_QuantumCircuits_EntanglingGates_WHT.ipynb) | CNOT, Toffoli, Walsh-Hadamard transform |
| 9 | [Demo9: Qiskit Introduction](../Demo9_Qiskit_Introduction.ipynb) | First circuits in Qiskit, simulators, visualisation |
| 10 | [Demo10: PennyLane Introduction](../Demo10_PennyLane_Introduction_Hands_On.ipynb) | Differentiable quantum circuits in PennyLane |
| 11 | [Demo11: Qiskit-PennyLane Synthesis](../Demo11_Qiskit_PennyLane_Synthesis_and_Comparison.ipynb) | Framework comparison, transpilation |
| 12 | [Demo12b: Oracles and Deutsch-Jozsa](../Demo12b_Qiskit_Oracles_Primitives_DJ.ipynb) | Quantum oracles, Deutsch-Jozsa algorithm |
| 13 | [Demo13b: Bernstein-Vazirani](../Demo13b_Bernstein_Vazirani_Qiskit.ipynb) | Interference-based query algorithms |
| 14 | [Demo14: Simon's Algorithm](../Demo14_Simons_Algorithm_Qiskit.ipynb) | Hidden subgroup problem, exponential speed-up |
| 15 | [Demo15: Grover's Algorithm](../Demo15_Grover_Qiskit.ipynb) | Amplitude amplification, quadratic speed-up |

---

*Part of the Quantum Computing Education Series by Trinexis, 2026.*  
*Notebook authored for classroom use. Feedback and corrections are welcome via the course repository.*
