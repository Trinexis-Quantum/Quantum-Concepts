# Notebook 6, Noise, Errors & Quantum Information Measures

> **Quantum Computing Education Series by Trinexis · Trinexis**
> *Hands-On Series*

![Python](https://img.shields.io/badge/Python-3.9%2B-blue?logo=python&logoColor=white)
![Qiskit](https://img.shields.io/badge/Qiskit-Compatible-6929C4?logo=ibm&logoColor=white)
![Jupyter](https://img.shields.io/badge/Jupyter-Notebook-F37626?logo=jupyter&logoColor=white)
![License](https://img.shields.io/badge/License-MIT-green?logo=opensourceinitiative&logoColor=white)

---

## Overview

Quantum computers are not perfect. Every real qubit is surrounded by an environment, stray electromagnetic fields, neighbouring atoms, vibrations in the substrate, that constantly tugs at it, leaking information outward and scrambling the delicate phase relationships that make quantum algorithms work. This notebook is your guided tour through exactly what that noise looks like mathematically, how to picture it geometrically, and how to measure the damage it does.

Think of a qubit's quantum state as an arrow (the Bloch vector) sitting inside a unit sphere. A perfect quantum gate is a rigid rotation of that arrow, it never changes the arrow's length. Noise is anything that *shrinks* the arrow toward the centre of the sphere, or drags it sideways, or pushes it toward the north pole. Part A of this notebook gives you five flavours of noise, bit-flip, phase-flip, depolarizing, amplitude damping, and phase damping, and shows you, through both algebra and vivid 3-D plots, exactly what each one does to the sphere. It is geometry, not just formalism.

Part B then hands you a toolkit of ten "rulers", purity, von Neumann entropy, fidelity, trace distance, coherence, mutual information, conditional entropy, entanglement entropy, relative entropy, and the Holevo quantity, each one answering a single plain-language question about your quantum state. By the end you will understand why a Stern–Gerlach experiment that *records* which path the spin took loses its interference pattern: that act of recording is precisely the dephasing channel, and dephasing is what kills coherence. The whole narrative of the course threads through this single notebook.

---

## Learning Objectives

After working through this notebook, you will be able to:

- **Describe** real qubit noise (bit-flip, phase-flip, depolarizing, amplitude damping, phase damping) in terms of Kraus operators and verify the CPTP completeness relation numerically.
- **Visualise** each noise channel as a geometric deformation of the Bloch sphere, an ellipsoid squeezed, shifted, or shrunk, and identify the deformation from its physical origin.
- **Derive** the Bloch-vector action of each channel from its Kraus operators and confirm agreement between the two representations in code.
- **Explain** the physical difference between T₁ relaxation (energy loss, longitudinal decay) and T₂ dephasing (phase loss, transverse decay), and state the fundamental bound T₂ ≤ 2T₁.
- **Connect** the dephasing channel to the Stern–Gerlach which-path experiment and explain precisely why which-path information destroys interference.
- **Compute** purity, von Neumann entropy, relative entropy, fidelity, trace distance, ℓ₁-coherence, relative-entropy coherence, mutual information, conditional entropy, entanglement entropy, and the Holevo χ for arbitrary qubit and two-qubit states.
- **Interpret** negative conditional entropy as a fingerprint of entanglement and distinguish it from classically correlated or product states.
- **Apply** the Holevo bound to explain why *n* qubits cannot transmit more than *n* classical bits, regardless of state dimension.
- **Choose** the right information measure as a diagnostic for a given question about noise, correlation, or entanglement.

---

## Background & Theory

### The Density Matrix and the Bloch Sphere (Quick Recap)

A single-qubit state is completely described by its **density matrix**

$$\rho = \frac{1}{2}\bigl(I + r_x X + r_y Y + r_z Z\bigr), \qquad |\mathbf{r}| \le 1,$$

where $\mathbf{r} = (r_x, r_y, r_z)$ is the **Bloch vector**. Pure states ($|\psi\rangle\langle\psi|$) live on the surface of the unit Bloch sphere; mixed states (statistical mixtures or noise-degraded states) live inside it. The maximally mixed state $I/2$ sits at the centre. A particularly clean identity links geometry to information:

$$\operatorname{Tr}(\rho^2) = \frac{1}{2}\bigl(1 + |\mathbf{r}|^2\bigr),$$

so **purity is literally the squared radius**, the single number that captures how far noise has dragged the state toward the centre.

---

### Quantum Noise Channels (CPTP Maps)

An isolated qubit evolves unitarily: $\rho \mapsto U\rho U^\dagger$. This is a *rigid rotation* of the Bloch ball, it preserves the radius and hence the purity. A qubit coupled to an environment evolves by a more general **completely positive, trace-preserving (CPTP) map**, which always admits a **Kraus (operator-sum) decomposition**:

$$\mathcal{E}(\rho) = \sum_i K_i \rho K_i^\dagger, \qquad \sum_i K_i^\dagger K_i = I.$$

The completeness relation $\sum_i K_i^\dagger K_i = I$ ensures $\operatorname{Tr}\mathcal{E}(\rho) = 1$ (probability is conserved). Unlike a unitary, a general CPTP map can **shrink** the Bloch ball, pure states become mixed, and information leaks irreversibly into the environment. The five canonical noise models and their Bloch-ball actions are:

| Channel | Kraus operators | Bloch action |
|---|---|---|
| Bit-flip (p) | $\sqrt{1-p}\,I,\ \sqrt{p}\,X$ | $r_x$ unchanged; $r_y, r_z \to (1-2p)r_{y,z}$ |
| Phase-flip (p) | $\sqrt{1-p}\,I,\ \sqrt{p}\,Z$ | $r_z$ unchanged; $r_x, r_y \to (1-2p)r_{x,y}$ |
| Bit-phase-flip (p) | $\sqrt{1-p}\,I,\ \sqrt{p}\,Y$ | $r_y$ unchanged; $r_x, r_z \to (1-2p)r_{x,z}$ |
| Depolarizing (p) | $\sqrt{1-3p/4}\,I,\ \sqrt{p/4}\{X,Y,Z\}$ | $\mathbf{r} \to (1-p)\mathbf{r}$ (isotropic shrink) |
| Amplitude damping ($\gamma$) | $K_0 = \begin{pmatrix}1&0\\0&\sqrt{1-\gamma}\end{pmatrix},\ K_1 = \begin{pmatrix}0&\sqrt{\gamma}\\0&0\end{pmatrix}$ | $r_{x,y} \to \sqrt{1-\gamma}\,r_{x,y};\quad r_z \to (1-\gamma)r_z + \gamma$ |
| Phase damping ($\lambda$) | $K_0 = \begin{pmatrix}1&0\\0&\sqrt{1-\lambda}\end{pmatrix},\ K_1 = \begin{pmatrix}0&0\\0&\sqrt{\lambda}\end{pmatrix}$ | $r_z$ unchanged; $r_{x,y} \to \sqrt{1-\lambda}\,r_{x,y}$ |

The decisive geometric fact is that **amplitude damping is the only channel whose image ellipsoid drifts off-centre**, it pulls toward the north pole $|0\rangle$ (the ground state), the signature of energy loss rather than mere phase scrambling.

---

### Decoherence and the T₁ / T₂ Picture

In experimental quantum hardware, two timescales dominate:

- **T₁ (longitudinal / relaxation time):** how long an excited qubit takes to decay to the ground state via energy loss. Described by amplitude damping with $\gamma(t) = 1, e^{-t/T_1}$; the excited-state population decays as $P(|1\rangle) = e^{-t/T_1}$.

- **T₂ (transverse / dephasing time):** how long a superposition state retains its phase coherence. Off-diagonal elements decay as $\rho_{01}(t) \sim e^{-t/T_2}$.

These are not independent. Relaxation also destroys coherence (every energy jump scrambles phase), so:

$$\frac{1}{T_2} = \frac{1}{2T_1} + \frac{1}{T_\varphi} \qquad \Longrightarrow \qquad \boxed{T_2 \le 2T_1}.$$

Here $T_\varphi$ is the **pure dephasing time**, coherence loss with no population change, the phase-damping channel.

---

### Quantum Information Measures

Once noise acts, we need diagnostic tools. Each of the ten measures below answers one sharp question:

**Purity:**
$$\gamma = \operatorname{Tr}(\rho^2) \in \bigl[\tfrac{1}{d},\, 1\bigr].$$

**Von Neumann entropy** (quantum generalisation of Shannon entropy):
$$S(\rho) = -\operatorname{Tr}(\rho \log_2 \rho) = -\sum_i \lambda_i \log_2 \lambda_i.$$
$S = 0$ for any pure state; $S = \log_2 d$ for the maximally mixed state. For a qubit it equals the binary entropy of $(1 \pm |\mathbf{r}|)/2$.

**Relative entropy** (quantum Kullback–Leibler divergence):
$$S(\rho \,\|\, \sigma) = \operatorname{Tr}\rho\log_2\rho, \operatorname{Tr}\rho\log_2\sigma \;\ge\; 0,$$
with equality iff $\rho = \sigma$. Diverges to $+\infty$ when $\rho$ has support outside the support of $\sigma$.

**Uhlmann Fidelity:**
$$F(\rho,\sigma) = \Bigl(\operatorname{Tr}\sqrt{\sqrt{\rho}\,\sigma\sqrt{\rho}}\Bigr)^2 \in [0,1].$$
$F = 1$ iff $\rho = \sigma$. For pure states, $F = |\langle\psi|\phi\rangle|^2$.

**Trace distance:**
$$T(\rho,\sigma) = \tfrac{1}{2}\operatorname{Tr}|\rho, \sigma|.$$
For qubits: $T = \frac{1}{2}|\mathbf{r}_\rho, \mathbf{r}_\sigma|$ (half the Euclidean Bloch-vector distance). Its operational meaning: the maximum probability of correctly distinguishing the two states in a single shot is $\frac{1}{2}(1+T)$ (Helstrom bound). The sandwich inequality holds:

$$1, \sqrt{F} \;\le\; T \;\le\; \sqrt{1-F}.$$

**Coherence (ℓ₁-norm):**
$$C_{\ell_1}(\rho) = \sum_{i \ne j} |\rho_{ij}|.$$
For a qubit this equals $\sqrt{r_x^2 + r_y^2}$, the length of the equatorial (transverse) component of the Bloch vector. It vanishes for diagonal states and is maximised ($= 1$) for $|+\rangle$.

**Mutual information** (total correlations, classical + quantum):
$$I(A\!:\!B) = S(A) + S(B), S(AB) \ge 0.$$

**Conditional entropy** (can go negative for entangled states!):
$$S(A|B) = S(AB), S(B).$$
Negative conditional entropy is a direct witness of entanglement.

**Entanglement entropy** (pure bipartite states):
$$E(|\psi\rangle_{AB}) = S(\rho_A) = S(\rho_B).$$

**Holevo quantity** (maximum extractable classical information):
$$\chi = S\!\Bigl(\sum_i p_i\rho_i\Bigr), \sum_i p_i S(\rho_i) \;\le\; \log_2 d.$$

---

### The Unifying Picture

Every noise channel *moves the Bloch vector*. Every information measure is a *geometric readout* of where that vector now sits. The dephasing channel specifically kills the transverse components $r_x, r_y$, it is precisely the mechanism by which a Stern–Gerlach experiment that records which $z$-path the spin took destroys interference. The course thesis, *indistinguishable alternatives interfere; probabilities are squared magnitudes of amplitudes*, finds its quantitative expression here.

---

## Prerequisites

| Topic | Where covered |
|---|---|
| Complex linear algebra (matrices, eigenvalues, trace) | Any undergraduate LA course |
| Quantum state vectors and Dirac notation | Notebooks 1–2 of this series |
| Density matrices, Bloch sphere, purity | Notebooks 3–4 of this series |
| Python / NumPy basics | Notebook 0 or equivalent |
| Familiarity with `matplotlib` 3-D plots | Helpful but not required |

No Qiskit installation is required; all simulations run on pure NumPy. A Google Colab session suffices. The interactive widget panel (Section B.12) requires `ipywidgets`, which is pre-installed in Colab.

---

## Notebook Walkthrough

### Cell 0, Title and Thesis Statement (Markdown)

The opening markdown cell introduces the notebook's two-part structure and restates the course's central thesis: *indistinguishable alternatives interfere; probabilities are squared magnitudes of amplitudes.* It previews Part A (noise as dynamics) and Part B (information measures as diagnostics), and plants the flag for the closing SG-cascade connection.

---

### Section 0, Setup and Reusable Toolkit

**Cell: Imports and Pauli matrices**
Imports `numpy`, `matplotlib`, and `mpl_toolkits`. Defines the four Pauli matrices (I, X, Y, Z), computational basis kets $|0\rangle$, $|1\rangle$, and the superposition kets $|+\rangle$, $|-\rangle$. Setting `np.random.seed(42)` makes all random demonstrations reproducible across different runs and machines, critical for classroom use.

**Cell: Carried-forward functions from Notebooks 3–4**
Re-defines `density_matrix_from_pure`, `rho_from_bloch`, `bloch_from_rho`, `purity`, and `born_probs` unchanged from earlier notebooks. This deliberate continuity is pedagogical: students see that the same toolkit serves both closed-system and open-system analysis.

**Cell: New matrix-function primitives**
Introduces `_spectral_apply`, `msqrt` (matrix square root via eigendecomposition), and `mlog2` (matrix base-2 logarithm using the $0\log 0 = 0$ convention). These are built from scratch rather than pulled from SciPy, keeping the spectral relationship explicit, you can *see* why the log of a density matrix makes sense. `apply_kraus` implements the operator-sum map $\sum_i K_i \rho K_i^\dagger$; `kraus_is_cptp` checks the completeness relation $\sum_i K_i^\dagger K_i = I$.

**Cell: Bloch-sphere renderer**
`draw_channel_image` draws a faint reference unit sphere and overlays the image ellipsoid produced by a channel characterised by scaling factors `lam` and offset `shift`. This renderer is called repeatedly in Section A.8 to produce the five-panel gallery.

**Note cell: Bloch conventions**
Reminds the reader that $\rho = \frac{1}{2}(I + \mathbf{r}\cdot\boldsymbol{\sigma})$, that purity equals $\frac{1}{2}(1+|\mathbf{r}|^2)$, and that this geometric identity will be the interpretive anchor for every channel in Part A.

---

### Part A, Noise Channels as Bloch-Sphere Deformations

**A.1, The Kraus language (Markdown + sanity-check cell)**
A unitary gate is a special CPTP map with a single Kraus operator $K_0 = U$. The cell applies a Hadamard channel to a state on the Bloch-sphere equator, confirms CPTP, and prints the rotated Bloch vector. *Why:* anchors the new formalism back to the familiar unitary case before introducing genuine noise.

**A.2, Bit-flip channel $\mathcal{E}_X$**
Implements `bit_flip_kraus(p)` and the analytic Bloch action `bit_flip_bloch`. Numerically confirms they agree for a representative mixed state and prints the CPTP check. The channel squashes the Bloch sphere into an ellipsoid pinched in the $y$-$z$ plane while preserving the $x$-axis, geometrically, a "flip around $X$" leaves the $X$-axis fixed.

**A.3, Phase-flip channel $\mathcal{E}_Z$ (dephasing)**
Implements `phase_flip_kraus(p)` and traces what happens to $|+\rangle$, the state of maximal $Z$-basis coherence, as $p$ runs from 0 to 0.5. At $p = 0.25$ half the coherence is gone; at $p = 0.5$ the off-diagonal elements are exactly zero and the state has become the classical mixture $I/2$. The notebook explicitly displays the off-diagonal element `|rho01|` at each step. *This section is the physical heart of the notebook*, it is where you see decoherence in action at the matrix level.

**A.4, Bit–phase-flip channel $\mathcal{E}_Y$**
A quick verification that the third Pauli channel ($Y$ error) preserves the $y$-axis while pinching the $x$-$z$ plane, completing the trio of Pauli channels.

**A.5, Depolarizing channel**
Implements the symmetric Pauli-twirl form. Confirms the isotropic Bloch action $\mathbf{r} \mapsto (1-p)\mathbf{r}$, the ball shrinks uniformly. The key structural point: depolarizing is the "worst-case" noise in quantum error-correction analyses because it has no preferred axis.

**A.6, Amplitude damping ($T_1$ relaxation)**
Implements the two-Kraus form with jump operator $K_1 = |0\rangle\langle 1|\sqrt{\gamma}$. Demonstrates the crucial asymmetry: $|0\rangle$ is the fixed point (ground state) and is unchanged, while $|1\rangle$ decays. The Bloch-vector shift $r_z \to (1-\gamma)r_z + \gamma$ is highlighted. A plot of excited-state population $P(|1\rangle) = e^{-t/T_1}$ versus $t/T_1$ gives the characteristic exponential $T_1$ decay curve students see in every superconducting-qubit paper.

**A.7, Phase damping ($T_2$ dephasing)**
Implements the two-Kraus "random phase kick" form. Shows algebraically and numerically that phase damping and the phase-flip channel are the same CPTP map written with different Kraus operators, demonstrating that Kraus representations are not unique. A plot of coherence $|\rho_{01}|$ and population $P(|1\rangle)$ versus $t/T_2$ side by side makes the $T_1$/$T_2$ distinction visceral: populations are frozen while coherence melts away.

**A.8, Five deformations side by side (Gallery plot)**
A single `plt.figure` with five 3-D subplots, one per channel at representative strength, each rendered by `draw_channel_image`. The key visual takeaway: amplitude damping is the only panel whose blue ellipsoid is *off-centre*, displaced toward the north pole, the geometric signature of energy loss versus pure phase scrambling.

**A.9, Decoherence IS the dephasing channel (SG closure)**
Returns to the Stern–Gerlach cascade from earlier notebooks. A spin prepared as $|+\rangle_x$ enters a region where the environment records the $z$-path. The reduced density matrix has its off-diagonals multiplied by $\kappa = \sqrt{1-\lambda}$, precisely the phase-damping action. A plot of interference visibility versus which-path distinguishability ($1, \kappa$) shows the complementary trade-off: as the environment records more, the fringe visibility falls to zero. *This is the course thesis made quantitative.*

---

### Part B, Quantum Information Measures

**Cell: Single-system measure toolkit**
Defines all six single-qubit measures, `von_neumann_entropy`, `relative_entropy`, `fidelity`, `trace_distance`, `l1_coherence`, `rel_entropy_coherence`, using `msqrt` and `mlog2`. All implementations are eigendecomposition-based, keeping the connection to the spectrum explicit.

**Cell: Bipartite helpers**
Implements `partial_trace` for 2-qubit systems (trace over either subsystem), then `mutual_information`, `conditional_entropy`, `entanglement_entropy`, and `holevo_chi`. A sanity check confirms $\operatorname{Tr}_B|\Phi^+\rangle\langle\Phi^+| = I/2$.

**B.1, Purity**
A four-row table checks $\gamma = \frac{1}{2}(1+|\mathbf{r}|^2)$ for $|0\rangle$, $|+\rangle$, a dephased $|+\rangle$, and $I/2$. Students see the formula is not just a definition, it is a geometric identity that holds for every state.

**B.2, Von Neumann entropy**
Verifies that $S(\rho)$ equals the binary entropy of the eigenvalue $(1+|\mathbf{r}|)/2$ for qubit states. Students watch entropy rise as dephasing and depolarizing noise increase, the entropy and purity dials moving in opposite directions.

**B.3, Relative entropy**
Demonstrates the asymmetry $S(\rho\|\sigma) \ne S(\sigma\|\rho)$, the zero condition $S(\rho\|\rho) = 0$, and the infinite case when supports do not match ($S(|0\rangle\langle 0| \| |1\rangle\langle 1|) = \infty$). The Klein inequality is verified numerically.

**B.4, Fidelity and trace distance**
Confirms $F(|+\rangle, |-\rangle) = 0$ and $T = 1$ for antipodal states; verifies $T = \frac{1}{2}|\mathbf{r}_\rho, \mathbf{r}_\sigma|$ for arbitrary qubit pairs; numerically checks the sandwich inequality $1, \sqrt{F} \le T \le \sqrt{1-F}$ for a randomly chosen pair.

**B.5, Coherence measures**
Tracks $C_{\ell_1}$ and $C_\text{rel}$ as $|+\rangle$ dephases. At $p = 0$, coherence is maximal; at $p = 0.5$, both measures hit zero, the state is now diagonal. The cell also confirms $C_{\ell_1} = \sqrt{r_x^2 + r_y^2}$ (equatorial Bloch length).

**B.6, Bipartite states and the Bell resource**
Demonstrates that the global Bell state $|\Phi^+\rangle$ is pure ($S_{AB} = 0$) yet each reduced state is maximally mixed ($S_A = 1$). Contrasts with a product state where $S_A = 0$. This tension, "the whole is more certain than any of its parts", is the qualitative definition of entanglement.

**B.7, Mutual information**
Computes $I(A\!:\!B)$ for the Bell state (= 2 bits), a product state (= 0), and a classically correlated mixture (= 1 bit). Confirms $I = 2S_A$ for pure bipartite states.

**B.8, Conditional entropy and quantum negativity**
Confirms $S(A|B) = -1$ bit for the Bell state, negative, the fingerprint of entanglement. The *coherent information* $I(A\!>\!B) = -S(A|B)$ is introduced as the quantity that governs quantum channel capacity.

**B.9, Entanglement entropy**
Studies the one-parameter family $\cos\theta|00\rangle + \sin\theta|11\rangle$ as $\theta$ sweeps from 0 (product) to $\pi/4$ (Bell state, 1 ebit). The Schmidt structure and the saturation at 1 ebit at $\theta = \pi/4$ are highlighted.

**B.10, Holevo quantity**
Plots $\chi$ versus the angle between two signal states: orthogonal states give $\chi = 1$ bit (fully readable); overlapping states reduce $\chi$ below 1 bit. The Holevo bound $\chi \le \log_2 d$ explains why you cannot transmit more than 1 classical bit per qubit regardless of which state you encode.

**B.11, Synthesis: four measures on the dephasing channel**
The capstone cell drives $|+\rangle$ through phase-flip at $p \in [0, 0.5]$ and simultaneously plots purity, von Neumann entropy, $\ell_1$ coherence, and trace distance from the original state. All four curves are monotone and consistent, a vivid single-axis picture of noise degradation read by four independent rulers.

**B.12, Interactive explorer (ipywidgets)**
A dropdown menu for channel type and a slider for strength let users dial any of the six channels and instantly read all five diagnostic measures. Falls back to a static two-row printout when `ipywidgets` is unavailable.

**B.13, One-screen reference table (Markdown)**
A ten-row summary table: measure name, the question it answers, its formula, and its Bloch-sphere / operational interpretation. Designed as a study card and quick reference for the exercises that follow.

---

### Exercises

Six graduated problems (★ to ★★★) reinforce the notebook's material:

| # | Stars | Topic |
|---|---|---|
| 1 | ★ | Verify isotropic scaling under depolarizing |
| 2 | ★ | Amplitude damping leaves $|0\rangle$ unchanged |
| 3 | ★★ | Numerical verification of $1-\sqrt{F} \le T \le \sqrt{1-F}$ |
| 4 | ★★ | Coherence is basis-relative |
| 5 | ★★★ | Entanglement family: verify $I = 2E$, $S(A|B) = -E$ |
| 6 | ★★★ | BB84 four-state Holevo and the no-cloning connection |

Fully worked solution cells are provided and commented.

---

## Key Takeaways

- **Noise is geometry.** Every qubit noise channel is a deformation of the Bloch sphere, Pauli channels pinch a plane, depolarizing shrinks uniformly, amplitude damping drags toward the north pole. Reading the geometry tells you the physics.

- **Amplitude damping is the odd one out.** It is the only channel whose image ellipsoid is off-centre. The shift $r_z \to (1-\gamma)r_z + \gamma$ encodes the asymmetry between ground and excited states, energy loss, not just phase scrambling.

- **T₂ ≤ 2T₁ always.** Every relaxation event also dephases, so the coherence time can never exceed twice the relaxation time. This inequality is hardware-universal.

- **Kraus representations are not unique.** Phase damping and the phase-flip channel produce identical CPTP maps; only their Kraus decompositions differ. The physics is in the map, not in any particular set of Kraus operators.

- **Decoherence is the dephasing channel.** The which-path Stern–Gerlach story is not a metaphor, it is exactly the phase-damping channel multiplying off-diagonal elements by $\kappa = \sqrt{1-\lambda}$. Interference visibility equals the surviving coherence.

- **Negative conditional entropy means entanglement.** A Bell pair has $S(A|B) = -1$ bit, a fact with no classical analogue. It signals that the global state contains more information than the sum of its parts, and that spare entanglement is "banked" for future protocols.

- **The Holevo bound is the qubits-vs-bits ceiling.** No matter how many states you use, $n$ qubits cannot deliver more than $n$ classical bits. Overlap between signal states costs you directly in extractable information, and that same overlap protects quantum key distribution.

---

## Real-World Applications

Understanding Noise and Quantum Information Measures is not just theoretical. Here is how it connects to active real-world problems and solutions:

- **Quantum Processor Benchmarking**: T₁ and T₂ times are the primary hardware metrics published by IBM, Google, and IonQ for every qubit on their quantum processors, directly informing which hardware to choose for a given task.
- **Quantum Error Correction**: The Kraus operator formalism is the mathematical foundation of surface codes, repetition codes, and all fault-tolerant quantum computing architectures.
- **Quantum Communication Channel Capacity**: The Holevo bound gives the fundamental limit on classical information transmitted through a quantum channel, used in quantum satellite link design.
- **Noise-Aware Circuit Compilation**: Quantum compilers (Qiskit Transpiler, pytket) use noise models built on the channels studied here to optimise circuit routing and minimise error rates on real hardware.
- **Quantum Error Mitigation (NISQ)**: Zero-noise extrapolation, probabilistic error cancellation, and Clifford data regression all rely on noise channel models studied in this notebook, enabling useful computation on today's noisy hardware.

---

## Further Reading & Citations

1. **Nielsen, M. A., & Chuang, I. L. (2000).** *Quantum Computation and Quantum Information.* Cambridge University Press., The canonical graduate text. Chapters 8–9 cover quantum noise and CPTP maps; Chapter 11 covers quantum information theory (entropy, fidelity, Holevo bound). ISBN 978-0-521-63503-5.

2. **Wilde, M. M. (2017).** *Quantum Information Theory* (2nd ed.). Cambridge University Press. arXiv:1106.1445., A rigorous modern treatment of all information measures discussed in Part B, with detailed proofs of the Holevo bound, data-processing inequality, and quantum channel capacities.

3. **Preskill, J. (1998).** *Lecture Notes for Physics 229: Quantum Information and Computation.* California Institute of Technology. Available at http://theory.caltech.edu/~preskill/ph229/, Chapter 3 on quantum channels and Chapter 5 on entropy are indispensable companions to this notebook.

4. **Haroche, S., & Raimond, J.-M. (2006).** *Exploring the Quantum: Atoms, Cavities, and Photons.* Oxford University Press., The definitive experimental account of decoherence and the quantum-to-classical transition; the cavity-QED experiments give concrete numbers for T₁ and T₂ in real systems. DOI: 10.1093/acprof:oso/9780198509141.001.0001.

5. **Baumgratz, T., Cramer, M., & Plenio, M. B. (2014).** Quantifying coherence. *Physical Review Letters*, 113(14), 140401. arXiv:1311.0275., The foundational paper establishing the resource-theoretic framework for coherence measures, including the ℓ₁-norm and relative-entropy coherence used in Section B.5.

6. **Terhal, B. M. (2015).** Quantum error correction for quantum memories. *Reviews of Modern Physics*, 87(2), 307–346. arXiv:1302.3428., Bridges exactly the noise models of Part A to the error-correction codes that are the next step in this course sequence; the depolarizing channel threshold analysis is covered in depth.

7. **Holevo, A. S. (1973).** Bounds for the quantity of information transmitted by a quantum communication channel. *Problemy Peredachi Informatsii*, 9(3), 3–11. (English translation: *Problems of Information Transmission*, 9(3), 177–183.), The original paper establishing the Holevo bound ($\chi$ quantity); accessible with the background from this notebook.

8. **Schlosshauer, M. (2007).** *Decoherence and the Quantum-to-Classical Transition.* Springer., A thorough and readable treatment of how dephasing channels connect to the classical limit; directly extends the SG discussion in Section A.9. DOI: 10.1007/978-3-540-35775-9.

---

## Related Notebooks

The full hands-on series for the series on Quantum Computing (Trinexis) covers:

| Notebook | Title | Link |
|---|---|---|
| 1 & 2 | Foundations: Qubits, Gates, and Circuits | [Handson1_2_Foundations.ipynb](./Handson1_2_Foundations.ipynb) |
| 3 | Density Matrices and Mixed States | [Handson3_Density_Matrices.ipynb](./Handson3_Density_Matrices.ipynb) |
| 4 | The Bloch Sphere and Visualisation | [Handson4_Bloch_Sphere.ipynb](./Handson4_Bloch_Sphere.ipynb) |
| 5 | Stern–Gerlach Cascade and Interference | [Handson5_SG_Cascade.ipynb](./Handson5_SG_Cascade.ipynb) |
| **6** | **Noise, Errors & Quantum Information Measures** | **This notebook** |
| 7 | Quantum Error Correction: 3-Qubit Codes | [Handson7_QEC_3Qubit.ipynb](./Handson7_QEC_3Qubit.ipynb) |
| 8 | CHSH Inequality and Bell Tests | [Handson8_CHSH_Bell.ipynb](./Handson8_CHSH_Bell.ipynb) |
| 9 | Quantum Teleportation | [Handson9_Teleportation.ipynb](./Handson9_Teleportation.ipynb) |
| 10 | Superdense Coding | [Handson10_Superdense_Coding.ipynb](./Handson10_Superdense_Coding.ipynb) |
| 11 | Grover's Search Algorithm | [Handson11_Grover.ipynb](./Handson11_Grover.ipynb) |
| 12 | Quantum Phase Estimation | [Handson12_QPE.ipynb](./Handson12_QPE.ipynb) |
| 13 | Shor's Factoring Algorithm | [Handson13_Shor.ipynb](./Handson13_Shor.ipynb) |
| 14 | Variational Quantum Eigensolver (VQE) | [Handson14_VQE.ipynb](./Handson14_VQE.ipynb) |
| 15 | Quantum Machine Learning Primer | [Handson15_QML.ipynb](./Handson15_QML.ipynb) |

---

<!-- SEO KEYWORDS
quantum noise, quantum error, CPTP map, Kraus operators, quantum channel, decoherence, dephasing channel, bit-flip channel, phase-flip channel, depolarizing channel, amplitude damping, T1 relaxation, T2 dephasing, Bloch sphere, density matrix, von Neumann entropy, quantum entropy, quantum information, fidelity, trace distance, quantum coherence, l1 coherence, relative entropy, mutual information, conditional entropy, entanglement entropy, Holevo bound, quantum information measures, quantum error correction, quantum computing education, Trinexis, quantum computing education series by Trinexisme, quantum channel capacity, quantum decoherence, purity, mixed state, quantum noise model, Stern-Gerlach, interference visibility, quantum tutorial, Jupyter notebook, NumPy quantum simulation, quantum optics
-->
