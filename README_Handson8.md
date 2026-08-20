# 🔗 Quantum Circuits, Entangling Gates & the Walsh–Hadamard Transform

[![Python 3.8+](https://img.shields.io/badge/Python-3.8%2B-3776AB?logo=python&logoColor=white)](https://python.org)
[![NumPy](https://img.shields.io/badge/NumPy-1.24%2B-013243?logo=numpy&logoColor=white)](https://numpy.org)
[![Matplotlib](https://img.shields.io/badge/Matplotlib-3.7%2B-11557c)](https://matplotlib.org)
[![Jupyter](https://img.shields.io/badge/Jupyter-Notebook-F37626?logo=jupyter&logoColor=white)](https://jupyter.org)
[![License: MIT](https://img.shields.io/badge/License-MIT-22c55e.svg)](https://opensource.org/licenses/MIT)
[![FDP IIT Roorkee](https://img.shields.io/badge/FDP-IIT%20Roorkee%202026-8b1a1a)](.)
[![No Qiskit Required](https://img.shields.io/badge/No%20Qiskit%20Required-Pure%20NumPy-blueviolet)](.)

**Hands-On 8 | Faculty Development Programme — Quantum Computing**
*IIT Roorkee, Department of Electronics & Communication Engineering, June–July 2026*

---

## Overview

Imagine you are stacking Stern–Gerlach magnets on a lab bench — first one magnet to split a beam, then a second to recombine it, then a third to filter again. You just built a quantum circuit. That is the central message of this notebook: a **quantum circuit** is nothing exotic. It is an ordered sequence of physically motivated operations ("gates") acting on qubits ("wires"), and everything you learned about single-qubit rotations and the Bloch sphere in earlier sessions now snaps together into a unified picture. This notebook is where all those pieces — bra-ket algebra, density matrices, purity, noise channels, and the gate zoo — get assembled into something you can actually *run*.

The first genuinely new idea introduced here is the **Walsh–Hadamard Transform (WHT)**: applying a Hadamard gate to every qubit simultaneously. One layer of $n$ parallel Hadamard gates creates a perfectly uniform quantum superposition over all $2^n$ computational basis states — the standard opening move of virtually every quantum algorithm you will encounter, from Deutsch–Jozsa to Grover to the Quantum Fourier Transform. The WHT has a beautiful classical cousin (the Walsh–Fourier basis used in spread-spectrum communications and signal processing), and understanding that connection gives you an immediate intuition for *why* it is useful.

The second new idea is **entangling gates**: two-qubit operations — CNOT, CZ, SWAP — and the three-qubit Toffoli gate. These are the ingredients that make quantum computing genuinely different from classical: they create correlations between qubits that cannot be explained by any local hidden-variable model. By the end of this notebook you will have built Bell states and GHZ states from scratch, confirmed their entanglement using the reduced-density-matrix purity test developed in Notebook 5, written a working statevector simulator in pure NumPy, and explored three carefully scaffolded exercises that take you from textbook identities to the exotic W-state entanglement class.

---

## Learning Objectives

After completing this notebook, you will be able to:

- **Define** a quantum circuit precisely as an ordered product of unitary operators acting on a tensor-product Hilbert space, and explain why the circuit diagram reads left-to-right while the matrix product is written right-to-left.
- **Derive** the action of the single-qubit Hadamard gate as the $z \leftrightarrow x$ Stern–Gerlach basis change, and visualise it as a 180° Bloch-sphere rotation about the $(\hat{x}+\hat{z})/\sqrt{2}$ axis.
- **Construct** the $n$-qubit Walsh–Hadamard Transform $H^{\otimes n}$ both via repeated Kronecker products and via the Sylvester recursive construction, and explain its role as the quantum analogue of the classical Walsh–Fourier transform.
- **Distinguish** local (separability-preserving) single-qubit gates from genuinely entangling two-qubit gates (CNOT, CZ, SWAP) and three-qubit gates (Toffoli), and explain why the distinction matters for information processing.
- **Build** the four Bell states and the three-qubit GHZ state from $|00\rangle$ and $|000\rangle$ respectively, using explicit matrix operations in NumPy.
- **Verify** entanglement using reduced-density-matrix partial traces and the purity diagnostic $\text{Tr}(\rho^2) = 0.5$ for maximally entangled qubits.
- **Implement** a minimal but complete statevector quantum circuit simulator — gates as matrices, wires as tensor factors, measurement as Born-rule sampling — entirely from scratch in NumPy.
- **Prove** the identities $\text{CZ} = (I \otimes H)\,\text{CNOT}\,(I \otimes H)$ and $\text{SWAP} = \text{CNOT}_{01}\,\text{CNOT}_{10}\,\text{CNOT}_{01}$ both analytically and numerically.
- **Construct** the three-qubit W state using controlled rotations and CNOTs, and explain why its entanglement class differs from the GHZ class under local operations.

---

## Background & Theory

### What is a Quantum Circuit?

A classical digital circuit routes bits through AND, OR, and NOT gates. A quantum circuit does the same thing, but for *qubits* and *unitary gates*. The key differences are:

1. **No copying.** The quantum no-cloning theorem forbids duplicating an arbitrary qubit state. Every wire persists independently.
2. **No deletion.** Every gate is reversible (unitary). There are no one-way gates.
3. **Superposition and interference.** A qubit can be in a superposition of $|0\rangle$ and $|1\rangle$, and circuits exploit interference between alternative paths to compute answers.

Formally, a quantum circuit on $n$ qubits computes:

$$|\psi_{\text{out}}\rangle = U_k \, U_{k-1} \cdots U_2 \, U_1 \, |\psi_{\text{in}}\rangle$$

Each $U_i$ is a unitary matrix. The state lives in $(\mathbb{C}^2)^{\otimes n}$, a vector space of dimension $2^n$. When two gates act on *separate* qubits simultaneously, the combined operation is their **tensor product** $U_A \otimes U_B$. When a gate acts on *multiple* qubits jointly and cannot be written as a tensor product, it is an **entangling gate** — the subject of the key section below.

### The Hadamard Gate: A Stern–Gerlach Basis Change

The Hadamard gate

$$H = \frac{1}{\sqrt{2}}\begin{pmatrix} 1 & 1 \\ 1 & -1 \end{pmatrix}$$

is the quantum circuit's most important single-qubit gate. Its physical meaning is exact: it is the operation that takes you from the $S_z$ eigenbasis $\{|0\rangle, |1\rangle\}$ to the $S_x$ eigenbasis $\{|+\rangle, |-\rangle\}$, precisely as rotating a Stern–Gerlach apparatus by 90° does in the lab:

$$H|0\rangle = \frac{|0\rangle + |1\rangle}{\sqrt{2}} = |+\rangle, \qquad H|1\rangle = \frac{|0\rangle - |1\rangle}{\sqrt{2}} = |-\rangle$$

On the Bloch sphere, $H$ is a 180° rotation about the diagonal axis $\hat{n} = (\hat{x} + \hat{z})/\sqrt{2}$. It swaps the $x$- and $z$-axes and negates $y$. Two crucial properties follow directly from this geometry: $H$ is **self-inverse** ($H^2 = I$, because two 180° rotations about the same axis return you to the start) and **unitary** ($H^\dagger H = I$). Sending a qubit through two consecutive Hadamards with no measurement in between is exactly the double-SG-magnet coherence experiment from Notebook 1: the alternatives recombine, and nothing net happens.

### The Walsh–Hadamard Transform

Applying $H$ simultaneously to all $n$ qubits gives the **Walsh–Hadamard Transform**:

$$H^{\otimes n} = \underbrace{H \otimes H \otimes \cdots \otimes H}_{n \text{ factors}}$$

Starting from $|0\rangle^{\otimes n}$, this produces the **uniform superposition**:

$$H^{\otimes n}|0\rangle^{\otimes n} = \frac{1}{\sqrt{2^n}} \sum_{x=0}^{2^n-1} |x\rangle$$

This is a vector with $2^n$ equal-amplitude components — simultaneously representing every $n$-bit string. This "quantum parallelism" is the starting point for nearly every quantum algorithm.

The matrix $H^{\otimes n}$ (up to the $1/\sqrt{2^n}$ normalisation) is the classical **Sylvester–Hadamard matrix**, built by the recursive doubling:

$$H^{\otimes n} = \frac{1}{\sqrt{2}}\begin{pmatrix} H^{\otimes(n-1)} & H^{\otimes(n-1)} \\ H^{\otimes(n-1)} & -H^{\otimes(n-1)} \end{pmatrix}$$

Each entry has the elegant closed form:

$$\left(H^{\otimes n}\right)_{xy} = \frac{(-1)^{x \cdot y}}{\sqrt{2^n}}, \qquad x \cdot y = \bigoplus_i x_i y_i \pmod{2}$$

where $x \cdot y$ is the bitwise inner product. Each row is a **Walsh function** — the same basis functions used in spread-spectrum radio coding (CDMA), fast Walsh–Fourier analysis, and Boolean function theory. This is no coincidence: Deutsch–Jozsa and related algorithms are literally performing a Walsh–Fourier transform on a quantum oracle.

> **Misconception to avoid.** "The uniform superposition already contains the answer — we have exponential speedup for free." This is wrong. A measurement on $H^{\otimes n}|0\rangle^{\otimes n}$ alone yields a uniformly random $n$-bit string, carrying no useful information. The speedup comes from the subsequent interference pattern created by an oracle and additional gates that *shape* the amplitudes before measurement.

### Entangling Gates

Local (single-qubit) gates act as $U_1 \otimes U_2 \otimes \cdots$. A fundamental theorem of quantum information states that **local unitaries cannot create entanglement from a product state**. To entangle qubits, you need a gate that genuinely couples two or more wires.

**CNOT (Controlled-NOT):**

$$\text{CNOT} = \begin{pmatrix} 1&0&0&0 \\ 0&1&0&0 \\ 0&0&0&1 \\ 0&0&1&0 \end{pmatrix}, \qquad \text{CNOT}|c,t\rangle = |c,\, t \oplus c\rangle$$

CNOT flips the *target* qubit if and only if the *control* qubit is $|1\rangle$. Critically, it acts *coherently on superpositions*: if the control is in $|+\rangle = (|0\rangle + |1\rangle)/\sqrt{2}$, the result is not a classically conditional flip but a genuinely entangled state.

**The Bell-state recipe:** One Hadamard followed by one CNOT turns $|00\rangle$ into the maximally entangled Bell state:

$$|00\rangle \xrightarrow{H \otimes I} \frac{|00\rangle+|10\rangle}{\sqrt{2}} \xrightarrow{\text{CNOT}} \frac{|00\rangle+|11\rangle}{\sqrt{2}} = |\Phi^+\rangle$$

The Hadamard creates *which-path ambiguity* on qubit 0; the CNOT then makes qubit 1's state *depend* on that ambiguity — the two qubits become one indivisible interference pattern. This is the multi-qubit generalisation of the SG cascade thread that runs through this entire course.

**CZ (Controlled-Z):** Applies a $-1$ phase only when both qubits are $|1\rangle$. Unlike CNOT, it is symmetric in its two qubits. The identity $\text{CZ} = (I \otimes H)\,\text{CNOT}\,(I \otimes H)$ shows that conjugating the CNOT target by Hadamards converts a bit-flip gate into a phase-flip gate.

**SWAP:** Exchanges the quantum states of two wires. Remarkably, three alternating CNOTs implement a full SWAP: $\text{SWAP} = \text{CNOT}_{01}\,\text{CNOT}_{10}\,\text{CNOT}_{01}$.

**Toffoli (CCNOT):** A three-qubit gate that flips the target only when *both* controls are $|1\rangle$. It implements a reversible AND gate — the classical universal gate — making it both a building block for error correction and a bridge between reversible classical and quantum computing.

### Entanglement Detection via Purity

Given a two-qubit state $|\psi\rangle$, we detect entanglement by computing the **reduced density matrix** of one qubit (tracing out the other):

$$\rho_A = \text{Tr}_B(|\psi\rangle\langle\psi|)$$

For a product state $|\psi\rangle = |\alpha\rangle \otimes |\beta\rangle$, the reduced state $\rho_A = |\alpha\rangle\langle\alpha|$ is pure: $\text{Tr}(\rho_A^2) = 1$. For the maximally entangled Bell state $|\Phi^+\rangle$, the reduced state is the maximally mixed state $\rho_A = I/2$: $\text{Tr}(\rho_A^2) = 0.5$. Purity $\text{Tr}(\rho^2)$ is therefore a continuous witness of entanglement, interpolating between 1.0 (separable) and $1/d$ (maximally entangled, where $d = 2$ for qubits).

### GHZ and W States: Two Inequivalent Entanglement Classes

For three qubits, two inequivalent families of maximally entangled states exist that *cannot* be converted into each other by any sequence of local operations and classical communication (LOCC):

- **GHZ state:** $|\text{GHZ}\rangle = (|000\rangle + |111\rangle)/\sqrt{2}$. Tracing out any single qubit leaves the remaining two in a *fully separable* (classically correlated) mixed state.
- **W state:** $|W\rangle = (|100\rangle + |010\rangle + |001\rangle)/\sqrt{3}$. Tracing out any single qubit leaves the remaining two in a *partially entangled* mixed state — entanglement is more "robust" in the W class.

These two families represent genuinely different ways that quantum information can be distributed across three parties, with different implications for quantum communication and error tolerance.

---

## Prerequisites

Before starting this notebook, you should be comfortable with the following (all covered in earlier notebooks in this series):

| Topic | Covered In |
|---|---|
| Dirac notation ($|\psi\rangle$, $\langle\phi|$, inner/outer products) | Hands-On 3 |
| Bloch sphere, single-qubit states as points on $S^2$ | Hands-On 3, 4 |
| Density matrices and partial trace | Hands-On 4 |
| Purity $\text{Tr}(\rho^2)$ as an entanglement witness | Hands-On 5 |
| Single-qubit gate zoo ($X$, $Y$, $Z$, $H$, $S$, $T$, rotation gates) | Hands-On 7 |
| Tensor products and Kronecker products (basic NumPy `kron`) | Hands-On 7 |
| Born rule and measurement collapse | Hands-On 3 |

**Software requirements:**
- Python 3.8+
- NumPy 1.24+
- Matplotlib 3.7+
- `ipywidgets` (optional; static fallback is provided automatically)
- No Qiskit, PennyLane, or SciPy required — this notebook uses pure NumPy throughout.

---

## Notebook Walkthrough

### Section 0 — Setup and Toolkit Reuse

The notebook opens by importing NumPy and Matplotlib and then re-loading the complete toolkit built across the series: `qubit_state`, `bloch_vector`, `rho_from_bloch`, `bloch_from_rho`, `purity`, `born_probs`, and `apply_kraus`. The single-qubit gate constants ($I$, $X$, $Y$, $Z$, $H$, $S$, $T$) from Hands-On 7 are also redefined inline.

**Why this structure?** Keeping the notebook self-contained means it can be run in any order, while the explicit re-derivation reinforces that every matrix in quantum computing is just a concrete $2\times2$ or $4\times4$ array of complex numbers. There is no black box.

The `ipywidgets` import is wrapped in a `try/except` with a `HAVE_WIDGETS` flag — a pattern introduced in Hands-On 2 that allows the notebook to convert to static HTML/PDF headlessly without interactive widget support.

### Section 1 — What Is a Quantum Circuit?

This section establishes the formal framework with a concrete physical anchor: the Stern–Gerlach cascade from Hands-On 1 is *already* a quantum circuit. Three helper functions are introduced:

- `kron_list(mats)`: computes the tensor product of a list of matrices left-to-right, handling any number of qubits cleanly.
- `embed_1q_gate(gate, target, n_qubits)`: lifts a single-qubit gate into the full $n$-qubit Hilbert space by tensoring with identity on all other wires.
- `ket(bitstring)`: constructs a computational basis state from a binary string such as `'010'`.

A sanity check demonstrates $(H \otimes I)|00\rangle = |+\rangle|0\rangle$ — the Hadamard acting only on qubit 0 leaves qubit 1 untouched, exactly as expected. This verifies that `embed_1q_gate` correctly implements the "implicit identity" on idle wires.

A **potential misconception** is flagged explicitly: circuit diagrams are not flowcharts. Unlike classical control-flow, there is no branching, no copying, and every wire is always present (with an implicit identity if no gate acts on it that step).

### Section 2 — The Hadamard Transform (Single Qubit)

This section gives three complementary views of the Hadamard gate:

1. **Algebraic:** The $2\times2$ matrix and its action on $|0\rangle$ and $|1\rangle$.
2. **Physical:** It is exactly the $z \leftrightarrow x$ Stern–Gerlach basis change — rotating the measurement apparatus by 90°.
3. **Geometric:** A 180° Bloch-sphere rotation about $(\hat{x}+\hat{z})/\sqrt{2}$, swapping the $z$- and $x$-axes and negating $y$.

Numerical verification confirms $H$ is unitary ($H^\dagger H = I$) and self-inverse ($H^2 = I$). A 3D Bloch-sphere plot shows the north pole ($|0\rangle$) rotating to the equator ($|+\rangle$). The optional interactive widget lets you drag any Bloch-sphere angle and watch $H$ act in real time — an effective pedagogical tool for building geometric intuition.

### Section 3 — The Walsh–Hadamard Transform ($n$ Qubits)

The WHT is built in two equivalent ways and verified to agree:

1. **Recursive (Sylvester) construction:** `walsh_hadamard(n)` doubles the matrix at each step using `np.block`, building the $2^n \times 2^n$ Hadamard matrix from the $1 \times 1$ seed `[[1]]`.
2. **Repeated Kronecker product:** `kron_list([H] * n)`.

Three bar charts (for $n = 1, 2, 3$) confirm that $H^{\otimes n}|0\rangle^{\otimes n}$ produces exactly uniform probability $1/2^n$ on every outcome — visually verifying the uniform superposition claim. A sign-pattern heatmap for $n = 3$ shows the checkerboard structure $(-1)^{x \cdot y}$, connecting WHT to the classical Walsh function basis. The caption makes explicit the link to spread-spectrum coding and Walsh–Fourier analysis.

A clearly labelled misconception callout addresses the "exponential speedup for free" fallacy: the uniform superposition alone carries no more information than a single uniformly random $n$-bit string — the interference from subsequent gates is what enables speedup.

### Section 4 — Entangling Gates

Four gates are defined as explicit matrices and verified to be unitary:

- **CNOT** ($4 \times 4$): flips target iff control is $|1\rangle$.
- **CZ** ($4 \times 4$): phases by $-1$ iff both are $|1\rangle$.
- **SWAP** ($4 \times 4$): exchanges the two wires.
- **Toffoli / CCNOT** ($8 \times 8$): flips target iff both controls are $|1\rangle$; constructed by swapping rows 6 and 7 of the $8 \times 8$ identity.

The **Bell-state circuit** is built gate by gate:
1. `embed_1q_gate(H, target=0, n_qubits=2)` creates superposition on qubit 0.
2. `CNOT` entangles qubit 1 with qubit 0.

The resulting statevector has amplitudes $[1/\sqrt{2}, 0, 0, 1/\sqrt{2}]^T$, confirming $|\Phi^+\rangle$. The `reduced_density_matrix` function computes the partial trace for the two-qubit case by reshaping $|\psi\rangle$ into a $2\times2$ matrix — an elegant trick that avoids writing a general partial-trace routine. Purity $= 0.5$ on both reduced states confirms maximal entanglement, directly reusing the diagnostic from Hands-On 5.

The **GHZ circuit** extends the construction to three qubits using `embed_2q_gate`, a general function that permutes basis indices to embed any two-qubit gate onto any (control, target) pair within an $n$-qubit register. Successive CNOTs from qubit 0 to qubits 1 and 2 spread the superposition across all three wires.

A **tunable entangling gate** family smoothly interpolates between the identity ($\alpha = 0$, no entanglement) and a fully entangling gate ($\alpha = \pi$, purity $= 0.5$). The resulting purity-vs-angle plot provides visual evidence that entanglement is a continuous, gate-parameter-dependent resource, not an all-or-nothing property.

### Section 5 — Circuit Composition, Simulation & Measurement

This section introduces the `QuantumCircuit` class: a minimal but complete statevector simulator with methods:

- `.h(wire)`, `.x(wire)`, `.z(wire)` — single-qubit gates, using `embed_1q_gate`.
- `.cnot(control, target)` — two-qubit entangling gate, using `embed_2q_gate`.
- `.probs()` — Born-rule probabilities via `born_probs`.
- `.sample(shots)` — Monte Carlo sampling using `np.random.default_rng`.
- `.draw()` — a Matplotlib circuit diagram renderer with standard CNOT symbols (filled dot for control, circled plus for target).

The class maintains an `ops` log of `(gate_name, wires)` tuples, allowing the diagram to be drawn directly from the same description used to evolve the state — keeping diagram and simulation in sync.

The Bell circuit is rebuilt using the simulator, drawn, and sampled for 1000 shots. The resulting histogram shows only `'00'` and `'11'` outcomes in a roughly 50/50 split — the hallmark of $|\Phi^+\rangle$ entanglement: each qubit individually looks maximally random, but their outcomes are perfectly correlated. This is the payoff of the whole section.

### Section 6 — Summary Table and Looking Ahead

A summary table maps each concept to its physical anchor and circuit role. The closing paragraph previews the next notebook (Bell/CHSH simulation), which builds directly on the Bell-state circuit, the purity witnesses, and the sampling machinery developed here.

### Section 7 — Exercises

Three carefully scaffolded exercises of increasing difficulty:

**Exercise 1 (★) — CZ from H and CNOT:** Verify numerically that $\text{CZ} = (I \otimes H)\,\text{CNOT}\,(I \otimes H)$. The scaffold returns `False` with a zero-matrix placeholder; the solution is a three-line function. The physical interpretation (conjugating the target by Hadamards converts a bit-flip into a phase-flip) is explained in the solution commentary.

**Exercise 2 (★★) — SWAP from three CNOTs:** Verify that $\text{SWAP} = \text{CNOT}_{01}\,\text{CNOT}_{10}\,\text{CNOT}_{01}$. The scaffold has the middle CNOT replaced by the identity; the solution inserts `CNOT_10`.

**Exercise 3 (★★★) — W State and Entanglement Structure:** Build $|W\rangle = (|100\rangle + |010\rangle + |001\rangle)/\sqrt{3}$ from $|000\rangle$ using controlled-$R_y$ rotations and CNOTs, with cascade angles $\theta_k = 2\arccos(1/\sqrt{n-k+1})$. The solution uses a controlled-$R_y$ helper and a sequence of five gates. After constructing $|W\rangle$, the exercise asks you to use the partial-trace tools to verify that the W and GHZ states are genuinely inequivalent entanglement classes: GHZ's two-qubit reduced state is fully separable, while W's retains partial entanglement.

---

## Key Takeaways

- **A quantum circuit is an ordered matrix product.** Diagrams read left-to-right; matrix multiplication applies right-to-left. Every idle wire carries an implicit identity.
- **The Hadamard gate is the $z \leftrightarrow x$ Stern–Gerlach rotation.** Self-inverse and unitary; its double application is the coherent recombination from the double-SG experiment.
- **The Walsh–Hadamard Transform opens almost every quantum algorithm** by creating a uniform superposition over $2^n$ states. Each row of the WHT matrix is a Walsh function, connecting quantum algorithms to classical signal processing.
- **Superposition alone does not create entanglement; a correlating gate must follow.** The sequence $H$ then CNOT is the canonical recipe: Hadamard creates which-path ambiguity, CNOT makes a second qubit *depend* on that ambiguity, producing $|\Phi^+\rangle$.
- **Purity of the reduced density matrix quantifies entanglement continuously.** $\text{Tr}(\rho_A^2) = 1$ means separable; $\text{Tr}(\rho_A^2) = 0.5$ means maximally entangled. Entanglement is not binary.
- **The CNOT, CZ, and SWAP gates satisfy exact algebraic identities** — $\text{CZ} = (I \otimes H)\,\text{CNOT}\,(I \otimes H)$ and $\text{SWAP} = \text{CNOT}_{01}\,\text{CNOT}_{10}\,\text{CNOT}_{01}$ — that reflect deep structural relationships among entangling operations.
- **GHZ and W states are inequivalent under local operations**, representing two fundamentally different ways quantum information can be distributed across three parties: GHZ entanglement is "fragile" (tracing out one qubit destroys all entanglement); W entanglement is "robust" (partial entanglement survives).

---

## Further Reading & Citations

1. **Nielsen, M. A., & Chuang, I. L. (2000).** *Quantum Computation and Quantum Information.* Cambridge University Press. — The canonical graduate textbook; Chapter 4 covers quantum circuits exhaustively, and Chapter 2 covers the tensor-product formalism and partial traces. [https://www.cambridge.org/9781107002173](https://www.cambridge.org/9781107002173)

2. **Preskill, J. (1998).** *Lecture Notes on Quantum Computation, Chapter 6: Quantum Circuits.* California Institute of Technology. — Freely available online; covers universality, the Solovay–Kitaev theorem, and the CNOT-based gate set. [http://theory.caltech.edu/~preskill/ph229/](http://theory.caltech.edu/~preskill/ph229/)

3. **Barenco, A., Bennett, C. H., Cleve, R., DiVincenzo, D. P., Margolus, N., Shor, P., Sleator, T., Smolin, J. A., & Weinfurter, H. (1995).** Elementary gates for quantum computation. *Physical Review A*, 52(5), 3457–3467. [https://doi.org/10.1103/PhysRevA.52.3457](https://doi.org/10.1103/PhysRevA.52.3457) — The foundational paper showing that any unitary can be decomposed into CNOT and single-qubit gates, and proving CNOT is a universal entangling gate.

4. **Deutsch, D., & Jozsa, R. (1992).** Rapid solution of problems by quantum computation. *Proceedings of the Royal Society of London A*, 439(1907), 553–558. [https://doi.org/10.1098/rspa.1992.0167](https://doi.org/10.1098/rspa.1992.0167) — The paper that introduced the WHT as a quantum computational tool in the Deutsch–Jozsa algorithm, the simplest demonstration of quantum advantage over deterministic classical algorithms.

5. **Dür, W., Vidal, G., & Cirac, J. I. (2000).** Three qubits can be entangled in two inequivalent ways. *Physical Review A*, 62(6), 062314. [https://doi.org/10.1103/PhysRevA.62.062314](https://doi.org/10.1103/PhysRevA.62.062314) — The paper that established the GHZ and W entanglement classes as inequivalent under LOCC, directly relevant to Exercise 3.

6. **Bell, J. S. (1964).** On the Einstein–Podolsky–Rosen paradox. *Physics*, 1(3), 195–200. [https://doi.org/10.1103/PhysicsPhysiqueFizika.1.195](https://doi.org/10.1103/PhysicsPhysiqueFizika.1.195) — The original paper demonstrating that entangled states (such as the Bell states built in this notebook) cannot be explained by any local hidden-variable theory; the CHSH game in the next notebook formalises this.

7. **Walsh, J. L. (1923).** A closed set of normal orthogonal functions. *American Journal of Mathematics*, 45(1), 5–24. [https://doi.org/10.2307/2387224](https://doi.org/10.2307/2387224) — The original 1923 paper introducing the Walsh functions that underpin the WHT's sign-pattern structure and the connection to classical spread-spectrum signal processing.

8. **Mermin, N. D. (2007).** *Quantum Computer Science: An Introduction.* Cambridge University Press. — A particularly accessible graduate introduction covering circuits and Bell inequalities with careful attention to exactly the conceptual points emphasised in this FDP course. [https://www.cambridge.org/9780521876582](https://www.cambridge.org/9780521876582)

---

<!-- 
SEO TAGS / KEYWORDS
===================
quantum circuits tutorial
entangling gates quantum computing
Walsh-Hadamard transform
CNOT gate matrix
Bell states quantum
GHZ state three qubits
W state entanglement class
quantum circuit simulator NumPy
statevector simulator Python
Hadamard gate Bloch sphere
partial trace purity entanglement
quantum tensor product Kronecker
controlled-NOT gate
CZ gate SWAP gate Toffoli
quantum Fourier transform Walsh functions
quantum circuit diagram Python
quantum gate universality
qubit entanglement detection
quantum computing FDP IIT Roorkee
quantum superposition interference
reduced density matrix
quantum algorithm Deutsch-Jozsa
GHZ W state inequivalent LOCC
quantum circuit education tutorial
quantum computing undergraduate course
-->

---

## Related Notebooks in This Series

The course follows a carefully sequenced progression. Each notebook builds directly on the previous ones:

| # | Notebook | Topic |
|---|---|---|
| 1–2 | `Demo1-2_Double_Slit_and_Stern_Gerlach_FDP.ipynb` | Double-slit experiment, Stern–Gerlach cascades, wave-particle duality |
| 3 | `Demo3_QMPostulates_BraKet_Bloch.ipynb` | QM postulates, bra-ket algebra, Bloch sphere geometry |
| 4 | `Demo4_BlochSphere_DensityMatrix.ipynb` | Density matrices, mixed states, Bloch-sphere geometry revisited |
| 5 | `Demo5_Purity_Coherence_Entanglement.ipynb` | Purity, decoherence, entanglement witnesses (tools reused here) |
| 6 | `Demo6_Noise_and_Information_Measures.ipynb` | Noise channels, Kraus operators, quantum entropy |
| 7 | `Demo7_Quantum_Gates_Demo.ipynb` | Single-qubit gate zoo: $X$, $Y$, $Z$, $H$, $S$, $T$, rotation gates |
| **8** | **`Demo8_QuantumCircuits_EntanglingGates_WHT.ipynb`** | **This notebook — multi-qubit circuits, entangling gates, WHT, Bell/GHZ states** |
| 9 | `Demo9_Qiskit_Introduction.ipynb` | Introduction to Qiskit: same circuits on real quantum hardware |
| 10 | `Demo10_PennyLane_Introduction_Hands_On.ipynb` | Introduction to PennyLane: differentiable quantum circuits |
| 11 | `Demo11_Qiskit_PennyLane_Synthesis_and_Comparison.ipynb` | Synthesis and comparison of frameworks |
| 12 | `Demo12b_Qiskit_Oracles_Primitives_DJ.ipynb` | Oracles, quantum parallelism, Deutsch–Jozsa on Qiskit |
| 13 | `Demo13b_Bernstein_Vazirani_Qiskit.ipynb` | Bernstein–Vazirani algorithm |
| 14 | `Demo14_Simons_Algorithm_Qiskit.ipynb` | Simon's algorithm and period finding |
| 15 | `Demo15_Grover_Qiskit_FDP.ipynb` | Grover's search algorithm |

---

*Notebook authored for the Faculty Development Programme in Quantum Computing, IIT Roorkee, June–July 2026. Runs entirely in Python with NumPy and Matplotlib — no quantum hardware or cloud access required.*
