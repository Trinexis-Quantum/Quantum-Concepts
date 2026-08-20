# 🔮 Hands-on 5 — Purity, Coherence, Entanglement & Multiqubit Systems

[![Python](https://img.shields.io/badge/Python-3.9%2B-blue?logo=python&logoColor=white)](https://www.python.org/)
[![Jupyter](https://img.shields.io/badge/Jupyter-Notebook-orange?logo=jupyter&logoColor=white)](https://jupyter.org/)
[![NumPy](https://img.shields.io/badge/NumPy-1.24%2B-013243?logo=numpy&logoColor=white)](https://numpy.org/)
[![Matplotlib](https://img.shields.io/badge/Matplotlib-3.7%2B-11557c)](https://matplotlib.org/)
[![ipywidgets](https://img.shields.io/badge/ipywidgets-8.0%2B-informational)](https://ipywidgets.readthedocs.io/)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](https://opensource.org/licenses/MIT)
[![FDP 2026](https://img.shields.io/badge/FDP-Quantum%20Computing%202026-purple)](.)

> **Faculty Development Programme · Quantum Computing · June–July 2026**
> File: `Demo5_Purity_Coherence_Entanglement.ipynb`

---

## Overview

Imagine you are trying to describe a coin. If you know the coin is heads-up, that is a *pure* state — complete information. But if someone flipped the coin behind a curtain and you don't know the result, you have a *mixed* state — a blend of possibilities. Quantum states carry this same distinction, but with a twist: a quantum superposition (like a coin that is genuinely both heads and tails at once) and a classical mixture (a coin you just haven't looked at yet) are physically different things, even though both sound like uncertainty. This notebook builds the mathematical tools — the **density matrix**, **purity**, **von Neumann entropy**, and **coherence** — that let you tell these situations apart precisely and quantitatively.

The second half of the notebook steps into one of the most counterintuitive corners of all of physics: **entanglement**. When two quantum systems interact, they can become correlated in a way that has no classical analogue whatsoever. Once entangled, measuring one instantly determines properties of the other — regardless of how far apart they are — yet no information travels between them faster than light. We build up to this from first principles: the **tensor product** for combining quantum systems, the **partial trace** for focusing on one subsystem, the **Schmidt decomposition** for revealing the structure of joint states, and **entanglement entropy** as a rigorous measure of how entwined two systems are.

A single unifying thread weaves through every section: *coherence, entanglement, and decoherence are three faces of one phenomenon.* When a quantum system interacts with its environment, it does not lose information — it *shares* that information with the environment through entanglement. The interference fringes that make quantum computing powerful disappear exactly because the environment has learned which path the system took. By the end of this notebook, you will be able to make that statement precise, numerical, and unavoidable.

---

## Learning Objectives

After completing this notebook, you will be able to:

- **Construct and validate** a density matrix from an ensemble of quantum states, and check the three necessary and sufficient conditions (Hermitian, positive semidefinite, unit trace).
- **Explain ensemble ambiguity**: why two physically different preparations can be completely indistinguishable by any measurement.
- **Compute purity** $\mathrm{Tr}(\rho^2)$ and connect it geometrically to the Bloch-vector radius, confirming the identity $\mathrm{Tr}(\rho^2) = \tfrac{1}{2}(1+|\mathbf{r}|^2)$ numerically.
- **Calculate and interpret** the von Neumann entropy $S(\rho) = -\mathrm{Tr}(\rho \log_2 \rho)$ as a measure of quantum ignorance.
- **Distinguish coherence from purity**: explain why coherence is basis-dependent whereas purity is intrinsic, and quantify coherence using the $\ell_1$-norm.
- **Simulate dephasing** as the decay of off-diagonal density matrix elements, and connect it to the washing-out of interference fringes.
- **Build two-qubit states** using the Kronecker (tensor) product and apply local operators on individual qubits.
- **Identify the four Bell states** and prove algebraically that they cannot be written as product states.
- **Apply the partial trace** to obtain a reduced density matrix and interpret what it means physically.
- **Use the Schmidt decomposition** (via SVD) to determine the Schmidt rank of a bipartite state and compute its entanglement entropy.
- **Articulate the decoherence mechanism**: show numerically that system coherence lost to an environment equals the entanglement entropy gained with that environment.
- **Verify wave–particle (coherence–distinguishability) duality** $C_{\ell_1}^2 + D^2 = 1$ computationally.

---

## Background & Theory

### The Density Matrix: One Object to Rule Them All

A quantum state vector $|\psi\rangle$ is sufficient when you have *complete* knowledge of the system. In practice, two sources of uncertainty arise simultaneously: the irreducible quantum randomness of measurement outcomes, and classical ignorance about which state was prepared. The **density matrix** $\rho$ handles both at once.

For an **ensemble** — a source that emits state $|\psi_i\rangle$ with probability $p_i$ — the density matrix is:

$$\rho = \sum_i p_i \, |\psi_i\rangle\langle\psi_i|, \qquad p_i \ge 0, \quad \sum_i p_i = 1.$$

A matrix is a valid density matrix if and only if it satisfies three conditions:
1. **Hermitian**: $\rho = \rho^\dagger$ (so eigenvalues are real and interpretable as probabilities).
2. **Positive semidefinite**: $\langle\phi|\rho|\phi\rangle \ge 0$ for all $|\phi\rangle$ (no negative probabilities).
3. **Unit trace**: $\mathrm{Tr}\,\rho = 1$ (total probability is one).

Every physically meaningful quantity — expectation values, measurement probabilities, post-measurement states — is encoded in $\rho$ via:

$$\langle A \rangle = \mathrm{Tr}(\rho A), \qquad p(m) = \mathrm{Tr}(P_m \rho).$$

A crucial and initially surprising fact is **ensemble ambiguity**: the identity matrix $\frac{1}{2}I$ is produced both by mixing $|0\rangle$ and $|1\rangle$ with equal probability *and* by mixing $|+\rangle$ and $|-\rangle$ with equal probability. No measurement, however clever, can distinguish these two histories. The density matrix contains everything physically meaningful; the "which ensemble prepared this?" question has no observable answer.

### Purity and the Bloch Ball

The **purity**

$$\gamma = \mathrm{Tr}(\rho^2) = \sum_i \lambda_i^2$$

is a single number summarising how mixed a state is. It lives in the range $[1/d, 1]$ for a $d$-dimensional system. For a qubit, there is a beautiful geometric identity. Writing the density matrix in the Pauli basis as $\rho = \frac{1}{2}(I + \mathbf{r}\cdot\boldsymbol{\sigma})$, a direct calculation gives:

$$\mathrm{Tr}(\rho^2) = \frac{1}{2}(1 + |\mathbf{r}|^2).$$

Purity is literally the squared Bloch radius. Pure states ($\gamma=1$) live on the surface of the Bloch sphere; mixed states ($\gamma < 1$) live inside; the maximally mixed state $\frac{1}{2}I$ sits at the centre ($\gamma = \frac{1}{2}$, $|\mathbf{r}|=0$).

A complementary measure is the **von Neumann entropy**:

$$S(\rho) = -\mathrm{Tr}(\rho \log_2 \rho) = -\sum_i \lambda_i \log_2 \lambda_i \quad \text{(bits)}.$$

$S=0$ for a pure state (complete knowledge); $S = \log_2 d$ for the maximally mixed state (maximal ignorance). The **linear entropy** $S_L = 1 - \mathrm{Tr}(\rho^2)$ is a computationally cheaper approximation that has the same zeros and the same peak.

### Coherence: the Off-Diagonal Part

In a chosen basis $\{|0\rangle, |1\rangle\}$, split $\rho$ into:

- **Populations** (diagonal): $\rho_{00}$ and $\rho_{11}$ — the classical probabilities of each outcome.
- **Coherences** (off-diagonal): $\rho_{01} = \rho_{10}^*$ — the amplitude of interference between $|0\rangle$ and $|1\rangle$.

Kill the off-diagonals and you are left with a classical probability distribution. The off-diagonals are precisely what produces interference fringes. A quantitative measure of coherence in a given basis is the **$\ell_1$-norm**:

$$C_{\ell_1}(\rho) = \sum_{i \neq j} |\rho_{ij}|.$$

**Coherence is basis-dependent.** The state $|+\rangle$ has $C_{\ell_1} = 1$ in the $Z$ basis (maximal off-diagonals) but $C_{\ell_1} = 0$ in the $X$ basis (it is an eigenstate there). Purity is invariant under basis changes; coherence is not.

**Dephasing** is the process by which coherences decay. The dephasing channel

$$\mathcal{E}_p(\rho) = \left(1 - \frac{p}{2}\right)\rho + \frac{p}{2} Z\rho Z$$

leaves populations unchanged while shrinking the coherences: $\rho_{01} \to (1-p)\,\rho_{01}$. In continuous time, $\rho_{01}(t) = \rho_{01}(0)\,e^{-t/T_2}$, the familiar $T_2$ decay of NMR and superconducting qubits.

### Tensor Products and Multiqubit Hilbert Spaces

Two qubits $A$ and $B$ live in $\mathcal{H}_A \otimes \mathcal{H}_B$, a four-dimensional space spanned by $\{|00\rangle, |01\rangle, |10\rangle, |11\rangle\}$. Numerically, the tensor product is implemented as the **Kronecker product** `np.kron`. An operator acting on qubit $A$ alone is $A \otimes I_B$; on qubit $B$ alone it is $I_A \otimes B$. The exponential growth of Hilbert space — $2^n$ dimensions for $n$ qubits — is the fundamental resource that quantum computers exploit.

### Entanglement and the Partial Trace

A pure two-qubit state is **separable** if it factorises as $|\psi\rangle = |a\rangle_A \otimes |b\rangle_B$. If no such factorisation exists, it is **entangled**. The canonical maximally entangled states are the four **Bell states**:

$$|\Phi^\pm\rangle = \frac{1}{\sqrt{2}}(|00\rangle \pm |11\rangle), \qquad |\Psi^\pm\rangle = \frac{1}{\sqrt{2}}(|01\rangle \pm |10\rangle).$$

If we only have access to qubit $A$, its state is the **reduced density matrix** obtained by tracing out $B$:

$$\rho_A = \mathrm{Tr}_B(\rho_{AB}), \qquad \mathrm{Tr}_B(|a_1\rangle\langle a_2| \otimes |b_1\rangle\langle b_2|) = |a_1\rangle\langle a_2|\,\langle b_2|b_1\rangle.$$

The key theorem binds everything together:

> **A pure bipartite state is entangled if and only if its reduced density matrix is mixed.**

For the Bell state $|\Phi^+\rangle$, the pair is in a perfectly definite pure state, yet $\rho_A = \frac{1}{2}I$ — maximally mixed. All information resides in the correlations between $A$ and $B$, not in either subsystem alone.

The **entanglement entropy** $S(\rho_A)$ quantifies how entangled the pair is: it is $0$ for product states and $\log_2 d$ (in bits: $1$ bit for two qubits) for maximally entangled Bell states.

### Schmidt Decomposition

Any pure bipartite state can always be written as:

$$|\psi\rangle = \sum_i \sqrt{\lambda_i}\,|i\rangle_A \otimes |i\rangle_B, \qquad \lambda_i \ge 0, \quad \sum_i \lambda_i = 1.$$

This is the **Schmidt decomposition**: a single aligned sum in appropriate local bases. The $\sqrt{\lambda_i}$ are **Schmidt coefficients** (the singular values of the $d_A \times d_B$ coefficient matrix $C_{ij}$). The **Schmidt rank** — the number of non-zero terms — determines entanglement:
- Schmidt rank 1 $\iff$ separable.
- Schmidt rank $> 1$ $\iff$ entangled.

The $\lambda_i$ are exactly the eigenvalues of $\rho_A$, so the entanglement entropy is $S = -\sum_i \lambda_i \log_2 \lambda_i$.

### Decoherence = Entanglement with the Environment

The synthesis of the notebook: a quantum system does not simply lose coherence — it *entangles* with the environment. When a system qubit $|+\rangle_S$ interacts with a detector qubit $|0\rangle_D$ via a controlled coupling that copies which-path information with strength $\theta$:

$$|+\rangle_S|0\rangle_D \;\longrightarrow\; \frac{1}{\sqrt{2}}\Big(|0\rangle_S|0\rangle_D + |1\rangle_S(\cos\theta\,|0\rangle_D + \sin\theta\,|1\rangle_D)\Big).$$

The system's coherence is $|\rho^S_{01}| = \frac{\cos\theta}{2}$, and the entanglement entropy with the detector is a monotonically increasing function of $\theta$. **What the system loses in coherence, it gains in entanglement with its environment.** This is the microscopic mechanism behind decoherence — and explains precisely why which-path measurement destroys interference in the double-slit experiment.

The **coherence–distinguishability duality**

$$C_{\ell_1}^2 + D^2 = 1, \qquad D = \sin\theta,$$

is a quantitative version of wave–particle duality, verified numerically to machine precision in this notebook.

---

## Prerequisites

| Requirement | Notes |
|---|---|
| **Linear algebra** | Matrix multiplication, eigenvalues, eigenvectors, SVD |
| **Complex numbers** | Modulus, conjugate, Hermitian conjugate |
| **Python & NumPy** | Array operations, `np.kron`, `np.linalg` |
| **Matplotlib basics** | `plt.plot`, `plt.subplots`, `mpl_toolkits.mplot3d` |
| **Notebooks 3 & 4** | Bra-ket notation, state vectors, Pauli matrices, the Bloch sphere, and the density matrix as introduced in `Demo3_QMPostulates_BraKet_Bloch.ipynb` and `Demo4_BlochSphere_DensityMatrix.ipynb` |
| **No Qiskit/PennyLane needed** | This notebook runs on NumPy + Matplotlib only; zero additional setup |

---

## Notebook Walkthrough

### Section 0 — Setup and Reusable Toolkit

The notebook begins by importing NumPy and Matplotlib, fixing a random seed (`np.random.seed(42)`) for reproducible classroom demonstrations, and defining the Pauli matrices ($X$, $Y$, $Z$) and the common single-qubit kets ($|0\rangle$, $|1\rangle$, $|+\rangle$, $|-\rangle$, $|{+i}\rangle$).

This upfront toolkit design is intentional: every later section imports from here, so students can study any section independently without re-running boilerplate. The functions `density_from_state`, `bloch_vector`, `purity`, `von_neumann_entropy`, `l1_coherence`, and `expectation` are the entire mathematical infrastructure for the notebook, and they are deliberately kept in pure NumPy so the notebook runs on any machine, including free Colab, with no environment setup.

### Section 1 — The Density Matrix, Revisited

**Why revisit what Notebook 4 introduced?** Because here the density matrix does heavier lifting. Section 1.1 formalises the ensemble interpretation and carefully distinguishes quantum uncertainty (inherent in any $|\psi\rangle$) from classical uncertainty (which state was prepared). Section 1.2 gives the validation check `is_valid_density` — students can probe what goes wrong when one condition fails (the "impostor" matrix with a negative eigenvalue is a concrete counterexample). Section 1.4 demonstrates ensemble ambiguity: the calculation shows that mixing $|0\rangle$ and $|1\rangle$ vs. mixing $|+\rangle$ and $|-\rangle$ both produce $\frac{1}{2}I$, identical to numerical precision. This surprises almost every student and sets up the key message that the density matrix — not any particular ensemble — is the physical state.

### Section 2 — Purity

Section 2.1 defines purity and establishes its range. Section 2.2 gives the geometric identity $\mathrm{Tr}(\rho^2) = \frac{1}{2}(1+|\mathbf{r}|^2)$ and verifies it over 5,000 randomly generated qubit density matrices — the maximum error is $3.3 \times 10^{-16}$, confirming it to machine precision. This Monte Carlo verification builds both confidence and a habit of always checking analytical results numerically.

Section 2.3 plots purity and von Neumann entropy as the state is linearly mixed from the pure $|+\rangle$ toward $\frac{1}{2}I$. Students see that purity falls from $1$ to $\frac{1}{2}$ while entropy climbs from $0$ to $1$ bit — they move in opposite directions, confirming their complementary roles as measures of mixedness.

Section 2.4 is the interactive slider demonstration: dragging $m$ from $0$ to $1$ shows the Bloch vector literally retracting toward the centre of the sphere. This is the visual "aha" moment that makes the geometry concrete.

### Section 3 — Coherence

Section 3.1 introduces the population/coherence decomposition of $\rho$. Section 3.2 delivers the crucial conceptual point: the same state $|+\rangle$ has $C_{\ell_1} = 1$ in the $Z$ basis but $C_{\ell_1} = 0$ in the $X$ basis, while its purity (= 1) is unchanged. The code confirms this by rotating the density matrix with a Hadamard gate. Students who conflate "being in a superposition" with "having coherence" encounter the right correction here: coherence is a statement about a basis, not about the state alone.

Section 3.3 simulates dephasing and plots two views: the coherence decaying to zero as $p \to 1$, and the Bloch vector collapsing from the equator to the $z$-axis in the $xy$-plane. The connection to the double-slit and Stern–Gerlach experiments is made explicit in the accompanying markdown: the off-diagonals *are* the interference fringes, and their decay *is* the fringe washout.

### Section 4 — Multiqubit Systems and the Tensor Product

The `kron` helper wraps `np.kron` for arbitrary numbers of factors. Three canonical demonstrations follow in a single cell: constructing the computational basis of two qubits, building a product state $|+\rangle|0\rangle$, and applying a local gate ($X \otimes I$) to verify it flips only qubit $A$. These are the building blocks for everything in Section 5, and getting students comfortable with the `kron` indexing convention (qubit $A$ is the "leftmost" factor) before entanglement is introduced prevents a major source of confusion.

### Section 5 — Entanglement

Section 5.1 defines separability and gives the algebraic proof (by contradiction on the coefficient constraints) that $|\Phi^+\rangle$ cannot be written as a product. All four Bell states are constructed numerically and their orthonormality is verified via $M^\dagger M = I$.

Section 5.2 introduces the `partial_trace` function — arguably the most important new function in this notebook. The implementation reshapes the $4 \times 4$ density matrix into a four-index tensor and traces over the appropriate indices, which is exactly what the mathematical definition says. A sanity check on a product state confirms that tracing out $B$ from $|+\rangle|0\rangle$ recovers $|+\rangle\langle+|$ exactly, with purity $1$.

Section 5.3 is the theoretical core: the theorem that pure bipartite entanglement is equivalent to a mixed reduced state. A comparison table shows that product states yield $\rho_A$ with purity 1 and entropy 0, while Bell states yield $\rho_A = \frac{1}{2}I$ with purity $\frac{1}{2}$ and entropy 1 bit.

Section 5.4 introduces the Schmidt decomposition via SVD. The coefficient matrix $C$ is reshaped from the state vector and passed to `np.linalg.svd`; the singular values are the Schmidt coefficients. Product states have one nonzero singular value; Bell states have two equal ones ($1/\sqrt{2}$ each).

Section 5.5 is the second interactive demonstration: the family $\cos\theta\,|00\rangle + \sin\theta\,|11\rangle$ interpolates continuously from product ($\theta=0$) to maximally entangled ($\theta=\pi/4$). Students watch the Schmidt weight bar chart transition from $[1, 0]$ to $[1/2, 1/2]$ while the entropy curve rises from $0$ to $1$ bit.

### Section 6 — Synthesis: Decoherence as Entanglement

This section closes the narrative arc of the entire FDP series. The controlled-rotation model makes the decoherence mechanism microscopic and calculable. The plot of system coherence vs. system-detector entanglement as $\theta$ varies from $0$ to $\pi/2$ is the single most important figure in the notebook: the two curves are complementary, and together they demonstrate that interference is not destroyed — it migrates into correlations with the environment.

### Section 7 — Exercises (★ to ★★★)

Six exercises with collapsible solutions scale from warm-up (E1: compute purity for a simple mixed state) through core practice (E3: partial trace of a Bell state; E4: entanglement entropy of an asymmetric state) to genuine synthesis (E5: construct a state with exactly 0.5 bits of entanglement by solving the binary entropy equation; E6: verify coherence–distinguishability duality numerically). Each solution is a self-contained cell, so students can attempt the exercise in a fresh cell before revealing the answer.

---

## Key Takeaways

- **The density matrix is the complete state.** Two different physical preparations can be experimentally indistinguishable if they produce the same $\rho$; the density matrix contains everything physically meaningful, and nothing more.
- **Purity is geometry.** The Bloch-vector radius *is* the purity: $\mathrm{Tr}(\rho^2) = \frac{1}{2}(1+|\mathbf{r}|^2)$. Pure states sit on the sphere surface; mixed states live inside.
- **Coherence is basis-dependent; purity is not.** Whether a state "has coherence" is a statement about which measurement you choose, not about the state alone. This is why choosing the right basis is a practical skill in quantum information.
- **Entanglement iff mixed reduced state.** For a pure bipartite state, entanglement is exactly equivalent to a mixed reduced density matrix. The entanglement entropy $S(\rho_A)$ is the precise, basis-independent measure.
- **Schmidt rank is the entanglement fingerprint.** A pure bipartite state is separable iff its Schmidt rank is 1; the singular values of the reshaped coefficient matrix reveal everything about the entanglement structure.
- **Decoherence is not information loss — it is information redistribution.** When a system loses coherence, it does not vanish; it becomes entanglement with the environment. The equation $C_{\ell_1}^2 + D^2 = 1$ is a quantitative version of wave-particle duality and encapsulates this redistribution exactly.
- **NumPy is enough.** The entire theoretical machinery of quantum information — density matrices, partial traces, SVD-based Schmidt decompositions, von Neumann entropy — is implementable in clean, readable NumPy code. Building it from scratch, rather than calling a library, is the surest path to genuine understanding.

---

## Further Reading & Citations

1. **Nielsen, M. A. & Chuang, I. L.** (2010). *Quantum Computation and Quantum Information* (10th anniversary ed.). Cambridge University Press. — The definitive textbook reference. Chapters 2 (density matrices, partial trace), 4 (quantum circuits), and 9 (distance measures, entropy) cover all topics in this notebook at greater depth.

2. **Wilde, M. M.** (2017). *Quantum Information Theory* (2nd ed.). Cambridge University Press. [arXiv:1106.1445](https://arxiv.org/abs/1106.1445) — Rigorous treatment of von Neumann entropy, quantum channels, and entanglement measures. Chapter 4 on the density operator and Chapter 11 on entanglement are especially relevant.

3. **Baumgratz, T., Cramer, M., & Plenio, M. B.** (2014). Quantifying coherence. *Physical Review Letters*, **113**(14), 140401. [arXiv:1311.0275](https://arxiv.org/abs/1311.0275) — The foundational paper establishing the resource theory of coherence, including the $\ell_1$-norm of coherence $C_{\ell_1}$ used throughout this notebook.

4. **Schlosshauer, M.** (2007). *Decoherence and the Quantum-to-Classical Transition*. Springer. — The authoritative monograph on decoherence. The microscopic model in Section 6 of this notebook is a simplified version of the environment-induced superselection framework developed here. Also see the accessible review: [arXiv:quant-ph/0312059](https://arxiv.org/abs/quant-ph/0312059).

5. **Englert, B.-G.** (1996). Fringe visibility and which-way information: an inequality. *Physical Review Letters*, **77**(11), 2154–2157. — The original proof of the coherence–distinguishability duality $V^2 + D^2 \le 1$ (Exercise E6 in this notebook is its numerical demonstration). A landmark result connecting wave-particle duality to quantum information.

6. **Preskill, J.** (2022). *Quantum Computation Lecture Notes* (Chapter 2: Foundations of Quantum Theory). California Institute of Technology. [Available online](http://theory.caltech.edu/~preskill/ph229/) — Freely available graduate lecture notes with a clear and modern treatment of density matrices, entanglement, and the Schmidt decomposition.

7. **Rieffel, E. G. & Polak, W. H.** (2011). *Quantum Computing: A Gentle Introduction*. MIT Press. — A more accessible entry point than Nielsen–Chuang, particularly strong on tensor products and multi-qubit systems. Chapter 5 covers entanglement and Bell states at a level compatible with this notebook.

8. **Werner, R. F.** (1989). Quantum states with Einstein-Podolsky-Rosen correlations admitting a hidden-variable model. *Physical Review A*, **40**(8), 4277. — The paper that introduced the distinction between entangled and separable mixed states, extending the concept beyond pure states. Historically important for understanding the limits of the purity-as-entanglement criterion.

---

<!--
SEO TAGS / KEYWORDS — for discoverability

quantum computing tutorial, density matrix python, purity quantum state, von neumann entropy, quantum coherence, l1 norm coherence, dephasing channel, qubit mixed state, Bloch sphere visualization, tensor product qubits, kronecker product numpy, quantum entanglement, Bell states, partial trace implementation, reduced density matrix, Schmidt decomposition, singular value decomposition quantum, entanglement entropy, decoherence quantum computing, wave particle duality, coherence distinguishability duality, quantum information theory, multiqubit systems, quantum superposition vs mixture, ensemble ambiguity, quantum education, Faculty Development Programme quantum, jupyter notebook quantum mechanics, numpy quantum computing, open quantum systems, quantum noise, T2 dephasing, quantum state tomography, quantum resource theory, maximally entangled state, quantum error correction basics, FDP 2026 quantum, quantum mechanics Python, quantum computing course material
-->

---

## Related Notebooks in This Series

| # | Notebook | Topic |
|---|---|---|
| 1–2 | [`Demo1-2_Double_Slit_and_Stern_Gerlach_FDP.ipynb`](./Demo1-2_Double_Slit_and_Stern_Gerlach_FDP.ipynb) | Double-slit experiment, Stern–Gerlach, wave-particle duality |
| 3 | [`Demo3_QMPostulates_BraKet_Bloch.ipynb`](./Demo3_QMPostulates_BraKet_Bloch.ipynb) | QM postulates, bra-ket notation, Bloch sphere |
| 4 | [`Demo4_BlochSphere_DensityMatrix.ipynb`](./Demo4_BlochSphere_DensityMatrix.ipynb) | Bloch sphere geometry, density matrix introduction |
| **5** | **`Demo5_Purity_Coherence_Entanglement.ipynb`** ← *you are here* | **Purity, coherence, entanglement, multiqubit systems** |
| 6 | [`Demo6_Noise_and_Information_Measures.ipynb`](./Demo6_Noise_and_Information_Measures.ipynb) | Quantum noise, quantum channels, information measures |
| 7 | [`Demo7_Quantum_Gates_Demo.ipynb`](./Demo7_Quantum_Gates_Demo.ipynb) | Single- and two-qubit gates |
| 8 | [`Demo8_QuantumCircuits_EntanglingGates_WHT.ipynb`](./Demo8_QuantumCircuits_EntanglingGates_WHT.ipynb) | Quantum circuits, entangling gates, Walsh–Hadamard transform |
| 9 | [`Demo9_Qiskit_Introduction.ipynb`](./Demo9_Qiskit_Introduction.ipynb) | Introduction to Qiskit |
| 10 | [`Demo10_PennyLane_Introduction_Hands_On.ipynb`](./Demo10_PennyLane_Introduction_Hands_On.ipynb) | Introduction to PennyLane |
| 11 | [`Demo11_Qiskit_PennyLane_Synthesis_and_Comparison.ipynb`](./Demo11_Qiskit_PennyLane_Synthesis_and_Comparison.ipynb) | Qiskit vs PennyLane synthesis and comparison |
| 12b | [`Demo12b_Qiskit_Oracles_Primitives_DJ.ipynb`](./Demo12b_Qiskit_Oracles_Primitives_DJ.ipynb) | Deutsch–Jozsa algorithm, oracles, Qiskit primitives |
| 13b | [`Demo13b_Bernstein_Vazirani_Qiskit.ipynb`](./Demo13b_Bernstein_Vazirani_Qiskit.ipynb) | Bernstein–Vazirani algorithm |
| 14 | [`Demo14_Simons_Algorithm_Qiskit.ipynb`](./Demo14_Simons_Algorithm_Qiskit.ipynb) | Simon's algorithm |
| 15 | [`Demo15_Grover_Qiskit_FDP.ipynb`](./Demo15_Grover_Qiskit_FDP.ipynb) | Grover's search algorithm |

---

*Prepared for the Faculty Development Programme in Quantum Computing, June–July 2026. Notebook designed to run on Google Colab or any local Jupyter environment with Python 3.9+, NumPy, Matplotlib, and ipywidgets.*
