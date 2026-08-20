# Quantum Gates - A Hands-On Demo 🔬

![Python](https://img.shields.io/badge/Python-3.9%2B-blue?logo=python&logoColor=white)
![Qiskit](https://img.shields.io/badge/Qiskit-not%20required-lightgrey?logo=ibm&logoColor=white)
![Jupyter](https://img.shields.io/badge/Jupyter-Notebook-orange?logo=jupyter&logoColor=white)
![License](https://img.shields.io/badge/License-MIT-green?logo=opensourceinitiative&logoColor=white)

---

## Overview

Imagine you are learning a new language. You have already studied the alphabet (qubits and state vectors), the grammar rules (Hilbert spaces and the Bloch sphere), and the punctuation (measurement and Born probabilities). **Quantum gates are the verbs** - the action words that let you build sentences, paragraphs, and eventually entire programs.

In classical computing, logic gates like AND, OR, and NOT transform bits deterministically. Quantum gates work similarly on qubits, but with one crucial difference rooted in physics: every quantum gate must be **reversible**. That means if you know the output and the gate, you can always recover the exact input. Classical AND is not reversible (output 0 could have come from three different inputs), but a quantum NOT gate always maps $|0\rangle \to |1\rangle$ and $|1\rangle \to |0\rangle$ with no information loss. This reversibility is not a design choice - it is a mathematical consequence of how quantum mechanics works. Gates are **unitary operators**: square matrices $U$ satisfying $U^\dagger U = I$, which mathematically encodes both norm preservation (total probability always sums to 1) and reversibility ($U^{-1} = U^\dagger$).

A beautiful way to visualize single-qubit gates is through the **Bloch sphere** - a unit sphere where every point on the surface represents a possible pure qubit state, the north pole being $|0\rangle$ and the south pole being $|1\rangle$. Applying a single-qubit gate is geometrically equivalent to **rotating this sphere**. The $X$ gate (quantum NOT) rotates 180° around the $x$-axis, flipping north to south. The Hadamard gate $H$ rotates to the equator, placing the qubit into an equal superposition. The rotation gates $R_x$, $R_y$, $R_z$ provide continuous, physically-motivated knobs corresponding to actually driving a qubit with a magnetic field or microwave pulse in a laboratory. Two-qubit gates like CNOT go beyond single-sphere rotations; they entangle qubits so that neither particle has a well-defined state on its own - the information lives only in their joint correlations. This notebook builds every concept from scratch using NumPy and Matplotlib, confirming each mathematical claim in executable code.

---

## Learning Objectives

By the end of this notebook, you will be able to:

- State the **defining property of a quantum gate** (unitarity, $U^\dagger U = I$) and verify it numerically in code.
- Derive and confirm the three major **consequences of unitarity**: probability (norm) preservation, reversibility ($U^{-1} = U^\dagger$), and inner-product preservation.
- Build and apply the standard **single-qubit gate zoo**: $I$, $X$, $Y$, $Z$, $H$, $S$, $T$, $R_x$, $R_y$, $R_z$.
- Understand single-qubit gates geometrically as **Bloch-sphere rotations** and read visualizations of their action.
- Express a quantum gate as **timed Hamiltonian evolution** $U = e^{-iHt}$ and implement the matrix exponential via eigendecomposition (no SciPy required).
- Compose gates in **series** (matrix product, right-to-left) and in **parallel** (tensor/Kronecker product), and quantify the physical meaning of **non-commutativity** using the Stern–Gerlach cascade analogy.
- Build the standard **two-qubit gates** (CNOT, CZ, SWAP) from scratch and verify their truth tables.
- Create a **Bell state** (maximally entangled two-qubit state) with the minimal H-then-CNOT circuit and confirm entanglement via coefficient matrix rank and reduced-state purity.
- Compare **quantum vs. classical logic**: reversibility, the Toffoli bridge to classical computation, and the no-cloning theorem.
- Complete graded exercises that deepen intuition and challenge you to extend the ideas independently.

---

## Background & Theory

### Single-Qubit Gates and Unitary Matrices

A single-qubit state is a unit vector in $\mathbb{C}^2$:

$$|\psi\rangle = \alpha|0\rangle + \beta|1\rangle, \quad |\alpha|^2 + |\beta|^2 = 1.$$

A **quantum gate** on one qubit is a $2 \times 2$ unitary matrix $U$. The fundamental requirement is:

$$U^\dagger U = U U^\dagger = I_2,$$

where $U^\dagger = (U^*)^T$ is the conjugate transpose. This single equation guarantees three things:

1. **Norm preservation:** $\|U|\psi\rangle\| = \|\psi\rangle\|$, so total probability stays at 1.
2. **Reversibility:** $U^{-1} = U^\dagger$, so every gate is perfectly undoable.
3. **Inner-product preservation:** $\langle U\phi | U\psi\rangle = \langle\phi|\psi\rangle$, so quantum distinguishability is never artificially increased.

### Bloch Sphere and Rotations

Any pure single-qubit state can be written as:

$$|\psi\rangle = \cos\frac{\theta}{2}|0\rangle + e^{i\phi}\sin\frac{\theta}{2}|1\rangle,$$

and mapped to a point $(\sin\theta\cos\phi,\, \sin\theta\sin\phi,\, \cos\theta)$ on the unit sphere. Single-qubit gates act as rotations of this sphere. The rotation gate family for rotation by angle $\theta$ about axis $\hat{n}$ is:

$$R_{\hat{n}}(\theta) = e^{-i\theta(\hat{n}\cdot\vec{\sigma})/2} = \cos\frac{\theta}{2}\,I - i\sin\frac{\theta}{2}\,(\hat{n}\cdot\vec{\sigma}),$$

where $\vec{\sigma} = (X, Y, Z)$ is the vector of Pauli matrices.

### The Pauli Gates

The three Pauli matrices are the most fundamental single-qubit operators:

$$X = \begin{pmatrix}0 & 1 \\ 1 & 0\end{pmatrix}, \quad Y = \begin{pmatrix}0 & -i \\ i & 0\end{pmatrix}, \quad Z = \begin{pmatrix}1 & 0 \\ 0 & -1\end{pmatrix}.$$

Each is simultaneously **unitary** and **Hermitian** ($P^\dagger = P$), which means each is its own inverse ($P^2 = I$). The $X$ gate is the quantum NOT - it flips $|0\rangle \leftrightarrow |1\rangle$. The $Z$ gate leaves $|0\rangle$ alone and applies a phase flip to $|1\rangle \to -|1\rangle$ (important for interference). The $Y$ gate combines both a bit flip and a phase flip.

### Hadamard Gate

The Hadamard gate is arguably the most important gate in quantum computing. It maps:

$$H = \frac{1}{\sqrt{2}}\begin{pmatrix}1 & 1 \\ 1 & -1\end{pmatrix}, \quad H|0\rangle = |{+}\rangle = \frac{|0\rangle + |1\rangle}{\sqrt{2}}, \quad H|1\rangle = |{-}\rangle = \frac{|0\rangle - |1\rangle}{\sqrt{2}}.$$

It is the basis-change operator between the $z$-basis and the $x$-basis, directly corresponding to what a Stern–Gerlach analyzer along the $x$-axis does to a spin prepared along $z$. Notably, $H^2 = I$, so $H$ is its own inverse.

### Phase Gates: S and T

The $S$ gate and $T$ gate are phase rotation gates:

$$S = \begin{pmatrix}1 & 0 \\ 0 & i\end{pmatrix} = \sqrt{Z}, \qquad T = \begin{pmatrix}1 & 0 \\ 0 & e^{i\pi/4}\end{pmatrix} = \sqrt{S}.$$

These gates leave $|0\rangle$ unchanged and apply a phase to $|1\rangle$. Unlike the Pauli gates and Hadamard, $S$ and $T$ are unitary but **not** Hermitian, so they are not self-inverse. The set $\{H, T, \text{CNOT}\}$ is **universal** for quantum computation - any unitary transformation can be approximated to arbitrary precision using only these three types of gates.

### Gates as Hamiltonian Evolution

In a physical quantum system, applying a gate means turning on a Hamiltonian $H$ for a carefully chosen time $t$. The relationship is:

$$U(t) = e^{-iHt/\hbar}, \quad \hbar = 1 \text{ (natural units)}.$$

For example, driving a qubit with $H_\text{drive} = \frac{\Omega}{2} X$ for time $t = \pi/\Omega$ (a "$\pi$-pulse") produces exactly the $X$ (NOT) gate, up to an irrelevant global phase. The notebook implements this matrix exponential via eigendecomposition - $H = V \Lambda V^\dagger$ gives $e^{-iHt} = V\, e^{-i\Lambda t}\, V^\dagger$ - making the physics explicit without relying on `scipy.linalg.expm`.

### Multi-Qubit Gates and Entanglement

Two qubits span a $4$-dimensional space with basis $\{|00\rangle, |01\rangle, |10\rangle, |11\rangle\}$. A two-qubit state vector has four amplitudes, and two-qubit gates are $4 \times 4$ unitary matrices. The **CNOT gate** (Controlled-NOT) is the workhorse:

$$\text{CNOT}|00\rangle = |00\rangle, \quad \text{CNOT}|01\rangle = |01\rangle, \quad \text{CNOT}|10\rangle = |11\rangle, \quad \text{CNOT}|11\rangle = |10\rangle.$$

It flips the target qubit if and only if the control qubit is $|1\rangle$. The **Bell state** (maximally entangled pair) is created by the minimal two-gate circuit:

$$|\Phi^+\rangle = \text{CNOT}\,(H \otimes I)\,|00\rangle = \frac{|00\rangle + |11\rangle}{\sqrt{2}}.$$

This state cannot be written as a product $|\psi_0\rangle \otimes |\psi_1\rangle$; its coefficient matrix $C_{ab} = \langle ab | \Phi^+\rangle$ has rank 2 (a product state would have rank 1). Equivalently, the reduced density matrix of either qubit alone is maximally mixed with purity $1/2$, even though the joint state is pure with purity $1$.

The **Toffoli gate** (CCNOT) extends CNOT to three qubits: it flips the third qubit if and only if both the first two are $|1\rangle$. Setting the third qubit to $|0\rangle$ and reading the output computes a **reversible AND**, bridging classical and quantum computation. The Toffoli gate is a permutation matrix of size $8 \times 8$ and is its own inverse.

### The No-Cloning Theorem

A direct consequence of inner-product preservation is the **no-cloning theorem**: there is no unitary $U$ satisfying $U(|\psi\rangle|0\rangle) = |\psi\rangle|\psi\rangle$ for all $|\psi\rangle$. The proof is simple: if cloning preserved inner products, we would need $\langle\phi|\psi\rangle = \langle\phi|\psi\rangle^2$ for any two states, which forces $\langle\phi|\psi\rangle \in \{0, 1\}$ - only orthogonal or identical states. General non-orthogonal states cannot be cloned.

---

## Prerequisites

| Topic | Where covered |
|---|---|
| Complex numbers and linear algebra (vectors, matrices, eigenvalues) | Any standard linear algebra course |
| Qubit state vectors, ket notation, the Bloch sphere | Notebooks 1–2 of this course |
| Density matrices, purity, and partial trace | Notebooks 3–4 of this course |
| Born rule, measurement, and quantum information measures | Notebooks 5–6 of this course |
| Python + NumPy basics | Any introductory Python tutorial |

**Software:** Python 3.9+, NumPy, Matplotlib. Optional: `ipywidgets` (for the live Bloch-sphere rotation widget in Section 10). No Qiskit or SciPy required - all matrix operations are implemented from first principles.

---

## Notebook Walkthrough

### Cell 1 - Title and Framing Markdown
The opening markdown establishes where this notebook sits in the course arc (after qubits, Bloch sphere, density matrices, and quantum information measures) and states the central thesis in one sentence: *a quantum gate is a unitary operator - a reversible, norm-preserving rotation of the quantum state.* It also anchors the course's running physical analogy: the Stern–Gerlach (SG) spin-measurement cascade will reappear throughout to ground abstract matrix algebra in laboratory intuition.

### Cell 2 - Learning Objectives
Eight numbered objectives frame exactly what will be demonstrated and verified. This cell is a contract with the reader: by the end, they will be able to check unitarity in code, understand Bloch-sphere rotations, derive a gate from a Hamiltonian, compose gates correctly, build a Bell state, and appreciate the Toffoli–no-cloning contrast with classical computation.

### Cell 3 - Section 0: Setup
Imports NumPy and Matplotlib, sets `np.random.seed(42)` for reproducible demos, and attempts to import `ipywidgets`. The graceful fallback (`HAVE_WIDGETS = False`) ensures the notebook runs correctly in headless environments (automated grading, `nbconvert`) as well as in live Colab. **Why no Qiskit or SciPy?** This is a deliberate pedagogical choice: implementing the matrix exponential via eigendecomposition keeps the Hamiltonian-to-gate relationship fully transparent rather than hidden inside a library black box.

### Cell 4 - Section 1: Reusable Toolkit
Defines and explains every helper function that will be used throughout:
- **State constructors:** `qubit_state(theta, phi)`, `KET0`, `KET1`, `normalize`.
- **Bloch geometry:** `bloch_vector`, `density_matrix_from_pure`, `bloch_from_rho`, `purity`.
- **Measurement:** `born_probs`, `expectation`.
- **Gate utilities (new this notebook):** `is_unitary`, `is_hermitian`, `dagger`, `expm_herm`.

The key new function is `expm_herm(H, t)`, which computes $e^{-iHt}$ by diagonalizing the Hermitian matrix $H$: compute eigenvalues $\lambda_k$ and eigenvectors $V$, then return $V \cdot \text{diag}(e^{-i\lambda_k t}) \cdot V^\dagger$. This makes the gate-as-evolution connection explicit and is used in Section 6.

### Cells 5–8 - Section 2: Unitarity and Its Consequences
This is the theoretical core of the notebook, structured as a numeric demonstration of three consequences of $U^\dagger U = I$.

**Section 2 (cells 5–6):** Builds the Hadamard as the running example, prints its matrix, and calls `is_unitary(H)`. Verifies that $H^\dagger H = I$ numerically.

**Section 2a (cells 7–8):** Creates an arbitrary input state `psi`, applies $H$, and checks that both the norm and the sum of Born probabilities are exactly 1 before and after. *This is why unitarity is the physical requirement for gates:* quantum mechanics demands that total probability is conserved at every moment of evolution.

**Section 2b (cells 9–10):** Applies $H$ to `psi` and then applies $H^\dagger$ to the result, showing exact recovery of the original state. Compares this to the classical AND gate, which destroys input information and cannot be reversed - a fundamental design distinction between classical and quantum logic.

**Section 2c (cells 11–12):** Computes $\langle\phi|\psi\rangle$ before and after applying $H$ to both states, confirming the inner product is preserved. This result is flagged as "the root of the no-cloning theorem," foreshadowing Section 9.

### Cells 13–17 - Section 3: The Single-Qubit Gate Zoo
**Section 3 (cells 13–14):** Defines all seven standard single-qubit gates - $I$, $X$, $Y$, $Z$, $H$, $S$, $T$ - and tabulates whether each is unitary, Hermitian, and self-inverse ($U^2 = I$). Key finding: $X$, $Y$, $Z$, $H$ are all three; $S$ and $T$ are unitary but neither Hermitian nor self-inverse. This table motivates the distinction between "Pauli-type" gates and "phase-type" gates.

**Section 3a (cells 15–16):** Applies every gate to both $|0\rangle$ and $|1\rangle$ and prints the output vectors. Because gates are linear, these two outputs completely specify the gate's action on every possible input. The printout highlights that $X$ performs a quantum NOT, $Z$ only changes the phase of $|1\rangle$ (leaves probabilities alone), and $H$ creates equal superpositions.

**Section 3b (cells 17–18):** Anchors the Hadamard to the SG experiment: $H|0\rangle = |{+}\rangle$, which when measured in the $z$-basis gives 50/50, exactly the result a Stern–Gerlach $x$-analyzer would produce. Confirms $H^2 = I$ (going $z \to x \to z$ via two Hadamards returns you to the original state).

### Cells 19–21 - Section 4: Bloch-Sphere Visualizations
Defines `draw_bloch` and `plot_gate_action` to render 3D Bloch spheres with input and output vectors in different colors. Calls these functions for the $X$ gate (180° flip from north to south pole) and the $H$ gate (north pole to the $+x$ equatorial point). The narration below the plots interprets the pictures: $X$ is literally a half-turn about the $x$-axis; $H$ takes $|0\rangle$ to the state that has 50/50 $z$-measurement probability, connecting the geometry back to the Stern–Gerlach intuition.

### Cells 22–26 - Section 5: Rotation Gates
**Section 5 (cells 22–23):** Defines $R_x(\theta)$, $R_y(\theta)$, $R_z(\theta)$ using closed-form expressions, then verifies they agree exactly with `expm_herm` for $\theta = 0.9$ rad. This cross-check proves that the closed-form rotation gate really is timed Hamiltonian evolution - the two independent derivations yield the same matrix.

**Section 5a (cells 24–25):** Sweeps $R_y(\theta)$ from $\theta = 0$ to $\pi$ starting at $|0\rangle$ and plots the continuous trajectory of the Bloch vector, which traces a geodesic arc from the north pole to the south pole. The point: the classical world has only four single-bit gates (identity, NOT, constant-0, constant-1), but single-qubit quantum gates form a **continuous infinity** parameterized by the 3D rotation group.

### Cells 27–29 - Section 6: Gates as Hamiltonian Evolution
Demonstrates the $\pi$-pulse: driving with $H_\text{drive} = \frac{\Omega}{2} X$ for time $t_\pi = \pi/\Omega$ using `expm_herm` yields a matrix that acts exactly like the $X$ gate (up to a global phase). Then shows that a half-pulse ($t = \pi/(2\Omega)$) gives a $\sqrt{\text{NOT}}$ gate, and applying it twice recovers the $X$ gate. This section makes the connection between laboratory microwave pulses and the abstract gate matrices completely concrete.

### Cells 30–32 - Section 7: Composing Gates and Non-Commutativity
**Series composition:** $A$ followed by $B$ on the same qubit corresponds to the matrix product $BA$ (right-to-left ordering). **Parallel composition:** gates on different qubits at the same time correspond to the Kronecker product $A \otimes B$. The notebook computes $X \cdot H$ and $H \cdot X$ and shows they differ, then computes the commutator $[X, Z] = XZ - ZX$ and finds it nonzero. The SG physical anchor is made explicit: $S_x$ and $S_z$ are incompatible observables because $[X, Z] \neq 0$, which is exactly why inserting an $x$-analyzer into an SG cascade scrambles a previously sharp $z$-value. Non-commutativity is not an abstract algebraic curiosity - it is the mathematical face of measurement incompatibility.

### Cells 33–38 - Section 8: Two-Qubit Gates and Entanglement
**Section 8 (cells 33–34):** Constructs the $4$-element two-qubit basis using `np.kron`, defines CNOT, CZ, and SWAP as explicit $4\times 4$ matrices, verifies all are unitary, and prints the CNOT truth table.

**Section 8a (cells 35–36):** Runs the Bell circuit: apply $H \otimes I$ to $|00\rangle$ (Hadamard on qubit 0 only), then apply CNOT. The result is $|\Phi^+\rangle = (|00\rangle + |11\rangle)/\sqrt{2}$. The notebook then checks entanglement by reshaping the 4-component amplitude vector into a $2\times 2$ coefficient matrix and computing its rank. Rank 2 means the state is entangled (rank 1 would mean separable/product).

**Section 8b (cells 37–38):** Computes the reduced density matrix of qubit 0 by tracing out qubit 1 using `partial_trace_qubit1`. Finds purity = $1/2$ for the reduced state despite the joint state having purity = $1$. This is the defining signature of entanglement: each particle alone is maximally uncertain, but the pair together is in a definite pure state. The information lives entirely in the correlations.

### Cells 39–44 - Section 9: Quantum vs. Classical Gates
**Section 9 overview (cell 39):** A markdown comparison table (Classical vs. Quantum) covering operand type, operation type, reversibility, cloning, number of distinct gates, determinism, and universal gate sets.

**Section 9a (cells 40–41):** Builds the $8\times 8$ Toffoli gate as a permutation matrix (swap rows 6 and 7 of the identity), verifies it is unitary and self-inverse, then prints its reversible-AND truth table. Setting the third bit (ancilla) to 0 and reading the output implements AND while keeping the inputs intact - this is how classical irreversible logic is embedded in reversible quantum circuits.

**Section 9b (cells 42–43):** Demonstrates the no-cloning theorem numerically. Computes $\langle 0 | {+}\rangle = 1/\sqrt{2}$ and $\langle 0 | {+}\rangle^2 = 1/2$, which differ. A cloning machine preserving inner products would require these to be equal - they are not, so no such machine can exist for both $|0\rangle$ and $|{+}\rangle$ simultaneously.

### Cells 45–47 - Section 10: Interactive Euler Decomposition Widget
Defines `show_composed(alpha, beta, gamma)` which builds $R_z(\gamma) R_y(\beta) R_x(\alpha)$ and renders the output Bloch vector. When `ipywidgets` is available (live Colab), three `FloatSlider` widgets let you sweep angles continuously and watch the Bloch vector move in real time. This is the Euler angle decomposition theorem made interactive: **any** single-qubit gate is reachable by choosing the right three angles. In headless mode a static example renders instead.

### Cells 48–60 - Section 11: Graded Exercises
Three exercises with collapsible `<details>` solution cells.

**Exercise 1 (★):** Fill in two lines to verify that $S$ is unitary, not Hermitian, and satisfies $S^2 = Z$. A warm-up to confirm understanding of the toolkit functions.

**Exercise 2 (★★):** Generate all four Bell states by running the same H-then-CNOT circuit starting from each of the four computational basis inputs $|00\rangle$, $|01\rangle$, $|10\rangle$, $|11\rangle$. Students identify $|\Phi^+\rangle$, $|\Psi^+\rangle$, $|\Phi^-\rangle$, $|\Psi^-\rangle$.

**Exercise 3 (★★★):** Numerically show that $R_x(\pi/2)$ then $R_z(\pi/2)$ gives a different final Bloch vector than $R_z(\pi/2)$ then $R_x(\pi/2)$, starting from $|0\rangle$. Compute the Frobenius norm of the commutator $\|[R_x, R_z]\|_F$ and connect the nonzero result to the Stern–Gerlach incompatibility of $S_x$ and $S_z$.

### Cell 61 - Section 12: Recap
A final markdown cell summarizes every concept demonstrated in code, connects the Bell circuit to the CHSH/Bell-inequality notebook and the teleportation/superdense-coding notebooks coming later in the course, and reminds the reader that $\{H, T, \text{CNOT}\}$ is the universal gate set that PennyLane and Qiskit compile down to. Closes with the invitation to go back, change the numbers, and break things.

---

## Key Takeaways

- **Unitarity is the single defining property** of a quantum gate: $U^\dagger U = I$. Everything else - reversibility, probability conservation, inner-product preservation - follows from this one equation.
- **Single-qubit gates are Bloch-sphere rotations.** The Pauli gates are 180° flips about their respective axes; $H$ rotates the $z$-axis onto the $x$-axis; $R_x$, $R_y$, $R_z$ provide continuously variable rotation angles. There is no classical single-bit analogue of this continuum.
- **A gate is physically a timed Hamiltonian pulse.** The matrix exponential $U = e^{-iHt}$ is the bridge between abstract linear algebra and actual laboratory control sequences. A $\pi$-pulse under $H = (\Omega/2)X$ is exactly the NOT gate.
- **Gate composition is non-commutative**, and this non-commutativity is not a mathematical accident - it directly mirrors the Stern–Gerlach result that measuring spin along $x$ then $z$ differs from $z$ then $x$.
- **Entanglement requires a two-qubit gate.** Single-qubit gates acting independently on product states can never create entanglement. The minimal entangler is two gates: $H$ on one qubit followed by CNOT.
- **The no-cloning theorem is a corollary of unitarity.** Because gates preserve inner products, a machine that cloned two non-orthogonal states would have to change their overlap - a contradiction. Unknown quantum states cannot be copied.
- **The Toffoli gate bridges classical and quantum computation.** Any classical Boolean circuit can be made reversible using Toffoli gates, making classical computation a special case of quantum computation (with all states restricted to basis vectors).

---

## Further Reading & Citations

1. **Nielsen, M. A. & Chuang, I. L.** (2010). *Quantum Computation and Quantum Information* (10th Anniversary Ed.). Cambridge University Press. - The standard graduate reference; Chapters 1–4 cover all gate topics in this notebook with full proofs.

2. **IBM Qiskit Textbook** - "Gates and Circuits." *Learn Quantum Computation using Qiskit*. https://learning.quantum.ibm.com/course/basics-of-quantum-information - A freely accessible, interactive treatment of the same material with Qiskit-based circuit diagrams and simulators.

3. **Gottesman, D.** (1997). *Stabilizer Codes and Quantum Error Correction* (Doctoral thesis). California Institute of Technology. arXiv:quant-ph/9705052. - Introduces the Clifford group (generated by $H$, $S$, and CNOT), providing the deeper algebraic structure underlying the standard gate set.

4. **Barenco, A., Bennett, C. H., Cleve, R., DiVincenzo, D. P., Margolus, N., Shor, P., Sleator, T., Smolin, J. A., & Weinfurter, H.** (1995). Elementary gates for quantum computation. *Physical Review A*, 52(5), 3457–3467. https://doi.org/10.1103/PhysRevA.52.3457 - The foundational paper proving that $\{H, T, \text{CNOT}\}$ is universal and giving explicit decompositions of arbitrary unitaries into these elementary gates.

5. **Solovay, R. M.** (1995, unpublished); **Kitaev, A. Yu.** (1997). Quantum computations: Algorithms and error correction. *Russian Mathematical Surveys*, 52(6), 1191–1249. - Establishes the Solovay–Kitaev theorem: the universal gate set $\{H, T, \text{CNOT}\}$ can approximate any gate to precision $\varepsilon$ with $O(\log^c 1/\varepsilon)$ gates.

6. **Wootters, W. K. & Zurek, W. H.** (1982). A single quantum cannot be cloned. *Nature*, 299, 802–803. https://doi.org/10.1038/299802a0 - The original no-cloning theorem paper (two pages); beautifully simple proof directly accessible after completing this notebook.

7. **Preskill, J.** (2018). Quantum Computing in the NISQ Era and Beyond. *Quantum*, 2, 79. arXiv:1801.00862. https://doi.org/10.22331/q-2018-08-06-79 - Provides modern context for why the gate model matters in near-term hardware, connecting the abstract gate zoo to actual device constraints.

---

## Related Notebooks

The complete Hands-On series for the Quantum Computing Education Series by Trinexis:

| # | Title | Link |
|---|---|---|
| 1 | Qubits and State Vectors | [Handson1_Qubits_StateVectors.ipynb](./Handson1_Qubits_StateVectors.ipynb) |
| 2 | The Bloch Sphere | [Handson2_Bloch_Sphere.ipynb](./Handson2_Bloch_Sphere.ipynb) |
| 3 | Density Matrices | [Handson3_Density_Matrices.ipynb](./Handson3_Density_Matrices.ipynb) |
| 4 | Quantum Information Measures | [Handson4_Quantum_Information_Measures.ipynb](./Handson4_Quantum_Information_Measures.ipynb) |
| 5 | Measurement and Born Rule | [Handson5_Measurement_Born_Rule.ipynb](./Handson5_Measurement_Born_Rule.ipynb) |
| 6 | Entanglement and Purity | [Handson6_Entanglement_Purity.ipynb](./Handson6_Entanglement_Purity.ipynb) |
| **7** | **Quantum Gates (this notebook)** | **[Handson7_Quantum_Gates_Demo.ipynb](./Handson7_Quantum_Gates_Demo.ipynb)** |
| 8 | Quantum Circuits | [Handson8_Quantum_Circuits.ipynb](./Handson8_Quantum_Circuits.ipynb) |
| 9 | Bell Inequalities and CHSH Test | [Handson9_Bell_Inequalities_CHSH.ipynb](./Handson9_Bell_Inequalities_CHSH.ipynb) |
| 10 | Quantum Teleportation | [Handson10_Quantum_Teleportation.ipynb](./Handson10_Quantum_Teleportation.ipynb) |
| 11 | Superdense Coding | [Handson11_Superdense_Coding.ipynb](./Handson11_Superdense_Coding.ipynb) |
| 12 | Quantum Fourier Transform | [Handson12_Quantum_Fourier_Transform.ipynb](./Handson12_Quantum_Fourier_Transform.ipynb) |
| 13 | Grover's Search Algorithm | [Handson13_Grovers_Search.ipynb](./Handson13_Grovers_Search.ipynb) |
| 14 | Quantum Error Correction | [Handson14_Quantum_Error_Correction.ipynb](./Handson14_Quantum_Error_Correction.ipynb) |
| 15 | Variational Quantum Eigensolver (VQE) | [Handson15_VQE.ipynb](./Handson15_VQE.ipynb) |

---

<!-- SEO Keywords:
quantum gates tutorial, unitary matrix quantum computing, Bloch sphere rotation, Pauli gates X Y Z,
Hadamard gate superposition, CNOT gate entanglement, Bell state preparation, quantum NOT gate,
reversible quantum computing, no-cloning theorem, Toffoli gate reversible AND, quantum circuit gates,
single qubit gates, two qubit gates, rotation gates Rx Ry Rz, Hamiltonian evolution gate,
matrix exponential quantum, quantum gate universality, S gate T gate phase gate, numpy quantum simulation,
quantum gates python, Quantum Computing Education Series by Trinexis, Stern-Gerlach gate analogy, gate non-commutativity,
quantum vs classical logic, tensor product quantum, Kronecker product gates, Bell inequality preparation,
quantum superposition gates, eigendecomposition matrix exponential, quantum computing undergraduate tutorial
-->
