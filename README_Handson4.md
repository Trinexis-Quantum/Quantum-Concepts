# 🌐 Hands-On 4: The Bloch Sphere & The Density Matrix

[![Python](https://img.shields.io/badge/Python-3.10%2B-blue?logo=python&logoColor=white)](https://www.python.org/)
[![Jupyter](https://img.shields.io/badge/Jupyter-Notebook-orange?logo=jupyter&logoColor=white)](https://jupyter.org/)
[![NumPy](https://img.shields.io/badge/NumPy-2.0.2-013243?logo=numpy&logoColor=white)](https://numpy.org/)
[![Matplotlib](https://img.shields.io/badge/Matplotlib-3.10.0-11557c?logo=matplotlib&logoColor=white)](https://matplotlib.org/)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
[![FDP](https://img.shields.io/badge/FDP-Quantum%20Computing%202026-purple)](.)

---

## Overview

Imagine you are trying to describe the position of a point on the surface of the Earth. You need exactly two numbers — latitude and longitude — no matter how complicated the terrain looks from the ground. Remarkably, a single qubit (the fundamental unit of quantum information) lives in a space that, after accounting for all physical redundancies, is also characterised by exactly two real numbers. Those two numbers correspond to a point on a sphere — the **Bloch sphere** — and this notebook is devoted entirely to understanding that sphere, exploiting it, and then extending it to richer situations where a sphere alone is no longer enough.

This notebook is the fourth in the FDP on Quantum Computing series. After establishing the five postulates of quantum mechanics, the Pauli matrices, and the algebra of state vectors in the previous notebooks, we now shift to a completely geometric and intuitive picture of single-qubit quantum mechanics. Every state, every quantum gate, every measurement outcome, and every notion of quantum noise finds a vivid visual counterpart on or inside the Bloch sphere. You will build the drawing tools from scratch in NumPy and Matplotlib, drive them with interactive sliders, and watch animations that make abstract unitary evolution feel almost tangible.

The second half of the notebook introduces the **density matrix**, which is the key to describing three of the most important phenomena in real quantum systems: statistical uncertainty in state preparation, decoherence caused by environmental coupling, and the reduced description of one partner in an entangled pair. The density matrix is not a replacement for state vectors — it is a generalisation that contains them as a special case — and this notebook gives you the tools and the geometric intuition (the Bloch ball, not just the Bloch sphere) to work with it confidently. Everything built here — the helper functions, the visualisation routines, the dephasing model — feeds directly into the next notebook on two-qubit states, Bell pairs, and the CHSH inequality.

---

## Learning Objectives

After completing this notebook, you will be able to:

- Explain, from a counting-of-degrees-of-freedom argument, why every single-qubit pure state maps to a unique point on a 2-sphere.
- Write down and derive the canonical Bloch parameterisation $|\psi\rangle = \cos\frac{\theta}{2}|0\rangle + e^{i\varphi}\sin\frac{\theta}{2}|1\rangle$ and explain the physical significance of each angle.
- Compute the **Bloch vector** $\vec{r} = (\langle\sigma_x\rangle, \langle\sigma_y\rangle, \langle\sigma_z\rangle)$ for any qubit state, both analytically and numerically.
- Identify the six cardinal states ($|0\rangle$, $|1\rangle$, $|{\pm}\rangle$, $|{\pm i}\rangle$) on the sphere and state the antipodal-orthogonal correspondence.
- Understand why qubits are **spinors** — why a physical $2\pi$ rotation brings $|\psi\rangle$ to $-|\psi\rangle$ rather than back to itself, and why $4\pi$ is needed for full periodicity.
- Describe any single-qubit gate as a **rigid rotation** of the Bloch sphere and use the rotation formula $R_{\hat{n}}(\alpha) = \cos\frac{\alpha}{2}I - i\sin\frac{\alpha}{2}(\hat{n}\cdot\vec{\sigma})$.
- Read off measurement probabilities along any axis directly from the geometry: $P(\pm 1\,|\,\hat{n}) = (1 \pm \hat{n}\cdot\vec{r})/2$.
- Reproduce the Stern–Gerlach cascade numerically and explain why an intermediate measurement erases prior information.
- Construct the **density matrix** of a pure or mixed state from its Bloch vector, and invert the process to recover $\vec{r}$ from $\rho$.
- Distinguish pure states ($|\vec{r}|=1$, on the surface) from mixed states ($|\vec{r}|<1$, interior) using the purity $\mathrm{Tr}(\rho^2)$.
- Demonstrate that the same density matrix can arise from infinitely many different classical mixtures of pure states.
- Apply the **dephasing channel** model and predict how it shrinks the Bloch vector toward the $z$-axis.

---

## Background & Theory

### The Qubit as a Point on a Sphere

A classical bit is either 0 or 1 — nothing in between. A qubit, by contrast, can be in any superposition $|\psi\rangle = \alpha|0\rangle + \beta|1\rangle$ where $\alpha$ and $\beta$ are complex numbers. At first glance this looks like four real parameters of freedom ($\mathrm{Re}(\alpha)$, $\mathrm{Im}(\alpha)$, $\mathrm{Re}(\beta)$, $\mathrm{Im}(\beta)$), which would place the qubit in a very high-dimensional space. But two physical constraints cut this down dramatically:

1. **Normalisation**: since $|\alpha|^2 + |\beta|^2 = 1$ (total probability must be 1), we lose one real parameter. This constrains the state to lie on a unit sphere in $\mathbb{C}^2 \cong \mathbb{R}^4$ — a 3-sphere $S^3$.

2. **Global phase irrelevance**: the states $|\psi\rangle$ and $e^{i\theta}|\psi\rangle$ (for any real $\theta$) are physically identical — every observable expectation value is unchanged by a global phase factor. Quotienting out this $U(1)$ redundancy removes one more real degree of freedom.

What remains is a 2-dimensional real manifold: a 2-sphere $S^2$. This is the **Bloch sphere**. The canonical way to coordinatise it uses the polar angle $\theta \in [0, \pi]$ and azimuthal angle $\varphi \in [0, 2\pi)$:

$$|\psi(\theta, \varphi)\rangle = \cos\frac{\theta}{2}\,|0\rangle + e^{i\varphi}\sin\frac{\theta}{2}\,|1\rangle$$

The factor of $\theta/2$ rather than $\theta$ is not a notational accident — it is the signature of a **spin-$\frac{1}{2}$ system** (a spinor). It means that a $2\pi$ rotation of the physical sphere corresponds to a $4\pi$ rotation of the quantum state; a single full turn takes $|\psi\rangle$ to $-|\psi\rangle$, not back to itself. Only $4\pi$ restores the state exactly. This is the mathematical statement that qubits are spinors, members of the fundamental representation of $SU(2)$ rather than $SO(3)$.

### The Bloch Vector

The three Pauli operators $\{\sigma_x, \sigma_y, \sigma_z\}$ form a complete traceless Hermitian basis for $2\times 2$ matrices. Their expectation values in the state $|\psi(\theta,\varphi)\rangle$ are:

$$\langle\sigma_x\rangle = \sin\theta\cos\varphi, \quad \langle\sigma_y\rangle = \sin\theta\sin\varphi, \quad \langle\sigma_z\rangle = \cos\theta$$

These are precisely the Cartesian coordinates of the unit vector at colatitude $\theta$ and azimuth $\varphi$. The **Bloch vector** $\vec{r} = (\langle\sigma_x\rangle, \langle\sigma_y\rangle, \langle\sigma_z\rangle)$ is the most compact, coordinate-system-independent summary of a qubit state. It is the bridge between the abstract Hilbert space and the tangible geometry of a sphere.

### Quantum Gates as Rotations

One of the most beautiful facts in quantum information is that every $2\times 2$ unitary matrix (up to a global phase) is a rotation of the Bloch sphere. Specifically, a rotation by angle $\alpha$ about an axis $\hat{n}$ corresponds to the gate:

$$R_{\hat{n}}(\alpha) = e^{-i\alpha\hat{n}\cdot\vec{\sigma}/2} = \cos\frac{\alpha}{2}\,I - i\sin\frac{\alpha}{2}\,(\hat{n}\cdot\vec{\sigma})$$

Standard gates are special cases: $X$ is a $\pi$-rotation about $\hat{x}$; $H$ (Hadamard) is a $\pi$-rotation about the $(\hat{x}+\hat{z})/\sqrt{2}$ axis; the $S$ and $T$ gates are $\pi/2$ and $\pi/4$ rotations about $\hat{z}$. This geometric picture makes it obvious, for example, that $H$ swaps the north and south poles (exchanging $|0\rangle$ and $|1\rangle$) while mapping the equator to the prime meridian.

### The Density Matrix

A state vector $|\psi\rangle$ fully specifies an isolated, perfectly-known quantum system. Real experiments, however, routinely deal with three kinds of imperfect knowledge:

- **Preparation uncertainty**: the source emits $|\psi_i\rangle$ with classical probability $p_i$, but we do not record which $i$ occurred (a thermal spin beam, a noisy channel).
- **Environmental coupling (decoherence)**: the qubit interacts with many uncontrolled environmental degrees of freedom, losing phase coherence over time.
- **Entanglement with another system**: if two qubits are in a joint entangled state, neither qubit individually has a pure state — only the pair does.

All three cases are handled by the **density operator** (density matrix):

$$\rho = \sum_i p_i\,|\psi_i\rangle\langle\psi_i|, \qquad \sum_i p_i = 1,\; p_i \geq 0$$

The density matrix is the most general description of a quantum state. Any valid $\rho$ is Hermitian ($\rho^\dagger = \rho$), positive semi-definite ($\langle\phi|\rho|\phi\rangle \geq 0$ for all $|\phi\rangle$), and has unit trace ($\mathrm{Tr}\,\rho = 1$). These three conditions are both necessary and sufficient.

The elegant link to the Bloch sphere is the **master formula**:

$$\rho = \frac{1}{2}(I + \vec{r}\cdot\vec{\sigma}) = \frac{1}{2}(I + r_x\sigma_x + r_y\sigma_y + r_z\sigma_z)$$

For a pure state, $|\vec{r}| = 1$ and $\rho$ is a projector. For a mixed state, $|\vec{r}| < 1$ and $\rho$ has full rank. The entire **Bloch ball** (sphere plus interior) is the space of all physically allowed single-qubit states. The centre ($\vec{r} = 0$) is the **maximally mixed state** $\rho = I/2$, which encodes total ignorance about the qubit.

The **purity** $\mathrm{Tr}(\rho^2) = \frac{1}{2}(1 + |\vec{r}|^2)$ is a scalar measure of how pure the state is: $1$ for a pure state, $\frac{1}{2}$ for the maximally mixed state.

### Dephasing: A First Model of Decoherence

The simplest quantum noise model is the **dephasing channel**, which destroys phase coherence without causing bit-flip errors:

$$\rho \;\mapsto\; (1-\lambda)\,\rho + \lambda\,Z\rho Z, \qquad \lambda \in [0,1]$$

In Bloch vector terms, this multiplies the transverse components by $(1-2\lambda)$ while leaving $r_z$ untouched:

$$(r_x, r_y, r_z) \;\mapsto\; \bigl((1-2\lambda)r_x,\,(1-2\lambda)r_y,\,r_z\bigr)$$

At $\lambda = 0$ nothing happens; at $\lambda = \frac{1}{2}$ the Bloch vector collapses onto the $z$-axis, leaving a classical probability distribution over $|0\rangle$ and $|1\rangle$ with no quantum coherence; at $\lambda = 1$ the channel has applied $Z$ with certainty (a phase flip), which restores a pure state on the opposite side of the $z$-axis.

---

## Prerequisites

| Requirement | Notes |
|---|---|
| **Python 3.10+** | NumPy 2.x and Matplotlib 3.10 are used. |
| **Jupyter environment** | Works in JupyterLab, Jupyter Notebook, or Google Colab (recommended for interactive widgets). |
| **NumPy, Matplotlib** | Standard scientific Python. Pre-installed on Colab. |
| **ipywidgets** | Pre-installed on Colab; in a local environment run `pip install ipywidgets`. |
| **Notebooks 1–3** | Familiarity with Dirac notation, the five postulates of QM, and the Pauli matrices (all covered in Demo1-2 and Demo3 of this series) will make the material flow naturally, though each section opens with a brief recap. |
| **Linear algebra** | Matrix multiplication, eigenvalues, trace, Hermitian adjoints. Undergraduate level. |
| **Complex numbers** | Polar form, modulus, argument. |

No prior quantum computing hardware or Qiskit/PennyLane experience is needed for this notebook.

---

## Notebook Walkthrough

### Section 0 — Setup

The notebook opens by importing **NumPy** for linear algebra, **Matplotlib** (including the `mpl_toolkits.mplot3d` extension) for 3-D rendering, `matplotlib.animation` for animated rotations, and **ipywidgets** for interactive sliders. A random seed is fixed so every run produces identical outputs for the stochastic simulation in Section 6.5. The Pauli matrices and computational basis vectors are then redefined here (rather than imported from an earlier notebook) so the notebook is **fully self-contained** — you can open it standalone on any platform.

> Why self-contained? Because different participants in a workshop may jump to different notebooks depending on their background. The brief duplication is a pedagogical choice, not carelessness.

### Section 1 — Why a Sphere? Counting Degrees of Freedom

Before drawing anything, the notebook builds the mathematical case for why the qubit lives on a sphere. Starting from a general $\mathbb{C}^2$ vector with four real parameters, two physical constraints — normalisation and global-phase irrelevance — reduce the independent degrees of freedom to exactly two. A numerical experiment demonstrates global-phase invariance concretely: a random normalised qubit and its arbitrarily phased version yield identical Pauli expectation values to machine precision.

> This section prevents a common confusion: students often believe a qubit "should" have a 4-D or even infinite-dimensional description because it is a complex vector. The counting argument shows why only two parameters are physically meaningful.

### Section 2 — The Canonical Parameterisation

The derivation proceeds step by step: write the amplitudes in polar form, apply normalisation to set the moduli as $\cos(\theta/2)$ and $\sin(\theta/2)$, factor out the global phase, and define the relative phase $\varphi$. The six cardinal states ($|0\rangle$, $|1\rangle$, $|{+}\rangle$, $|{-}\rangle$, $|{+i}\rangle$, $|{-i}\rangle$) are computed numerically from $(\theta, \varphi)$ pairs and printed in Dirac notation using the `show_state` helper.

> The factor of $\theta/2$ is highlighted here but explained more deeply in Section 6.1 — a deliberate pedagogical delay that lets the student use the formula before understanding its deeper meaning.

### Section 3 — The Bloch Vector

The notebook derives analytically that $\langle\sigma_x\rangle = \sin\theta\cos\varphi$, $\langle\sigma_y\rangle = \sin\theta\sin\varphi$, $\langle\sigma_z\rangle = \cos\theta$, and then verifies numerically across a grid of 117 $(\theta,\varphi)$ pairs that the numerical and analytical Bloch vectors agree to $2.5 \times 10^{-16}$ (floating-point noise floor). The key `bloch_vector` function is introduced here and used throughout the notebook.

### Section 4 — A First Picture of the Bloch Sphere

The reusable `draw_bloch_sphere` function is built in this section. It renders a wire-frame sphere, labels the six cardinal states, draws the Cartesian axes in distinct colours, and overlays any number of labelled vectors. The design emphasises reusability: the same function powers every visualisation from Section 4 onward, including the interactive widgets and the animations.

### Section 5 — Interactive Bloch Sphere

The first slider widget appears here. Moving $\theta$ and $\varphi$ simultaneously updates four panels: the 3-D Bloch vector on the sphere, the state in Dirac notation, all three Pauli expectation values, and the Born-rule probabilities $P(|0\rangle) = \cos^2(\theta/2)$ and $P(|1\rangle) = \sin^2(\theta/2)$. The embedded "Try it" prompt asks students to hold $\varphi$ fixed and sweep $\theta$, then hold $\theta = \pi/2$ (the equator) and sweep $\varphi$ — a guided discovery of which quantity is invariant under each variation.

### Section 6 — Six Critical Properties

This is the conceptual heart of the notebook, split into five subsections.

**6.1 — The $\theta/2$ and spinors.** The 2$\pi$ sign-flip is demonstrated numerically: $R_z(2\pi)|{+}\rangle = -|{+}\rangle$ but $R_z(4\pi)|{+}\rangle = +|{+}\rangle$. The key clarification — that $-|\psi\rangle$ is physically the same state because it is only a global phase — is stated explicitly to prevent the common misconception that the sign flip is a physical observable.

**6.2 — Antipodal states are orthogonal.** An interactive widget shows a state $|\psi\rangle$ and its antipodal partner $|\psi^\perp\rangle$ simultaneously. The inner product $\langle\psi|\psi^\perp\rangle$ and the sum $\vec{r} + \vec{r}^\perp$ are displayed numerically, both confirming the relationship. This is a direct consequence of the half-angle: states that are 90° apart in Hilbert space are 180° apart on the Bloch sphere.

**6.3 — Gates as rotations.** The general rotation formula is stated and verified by showing that $R_x(\pi) = -iX$ (differing only by an unobservable global phase). An interactive widget lets the student select any of ten gates (I, X, Y, Z, H, S, T, $R_x(\pi/2)$, $R_y(\pi/2)$, $R_z(\pi/2)$) and any initial state, and displays both vectors on the sphere with the rotation visible.

**6.3b — Animation of continuous rotation.** Rather than showing a gate's action as an instantaneous jump, the animation sweeps $\alpha$ from $0$ to its final value, drawing the arc traced by the Bloch vector. The specific example shown is the Hadamard gate acting on $|0\rangle$ — the vector starts at the north pole and arcs along the $xz$-plane great circle to $|{+}\rangle$.

**6.4 — Measurement along an arbitrary axis.** The Born-rule formula for a general Hermitian observable $\hat{n}\cdot\vec{\sigma}$ is presented geometrically as a projection: $P(\pm 1|\hat{n}) = (1 \pm \hat{n}\cdot\vec{r})/2$. An interactive widget with four sliders (two for the state, two for the measurement axis) lets the student explore the three special cases: parallel (certain outcome), perpendicular (50/50 split), and antiparallel (certain opposite outcome).

**6.5 — Stern–Gerlach cascade on the Bloch sphere.** This section closes the loop with Demo 1-2. The cascade $|0\rangle \to \text{measure } x \to \text{measure } z$ is simulated with 20,000 shots using the `measure_once` collapse function. The final $z$-statistics come out 50/50, confirming that the intermediate $x$-measurement destroys the original $z$-information — the Stern–Gerlach erasure observation that historically motivated the quantum postulates.

### Section 7 — From State Vectors to Density Matrices

The conceptual motivation is stated clearly before the mathematics: three real situations (statistical ensembles, entangled subsystems, decoherence) cannot be described by a single ket. The density matrix is introduced as the generalisation, and its three defining properties (Hermitian, positive semi-definite, unit trace) are demonstrated numerically for both a pure state ($\rho = |{+}\rangle\langle{+}|$) and an incoherent mixture ($0.7|0\rangle\langle 0| + 0.3|1\rangle\langle 1|$). A key observation is highlighted in the output: the eigenvalues of a mixed-state $\rho$ are exactly the classical mixing probabilities.

### Section 8 — The Master Formula $\rho = \frac{1}{2}(I + \vec{r}\cdot\vec{\sigma})$

The derivation uses the fact that $\{I, \sigma_x, \sigma_y, \sigma_z\}$ is a complete real basis for $2\times 2$ Hermitian matrices. The trace condition fixes the coefficient of $I$ to $\frac{1}{2}$, and the remaining three coefficients are exactly the Bloch components. Two key functions are introduced: `rho_from_bloch` (construct $\rho$ from $\vec{r}$) and `bloch_from_rho` (extract $\vec{r}$ via $r_k = \mathrm{Tr}(\rho\sigma_k)$). A round-trip test confirms that the two operations are mutual inverses to machine precision.

### Section 9 — Pure States on the Surface, Mixed States Inside

The purity formula $\mathrm{Tr}(\rho^2) = \frac{1}{2}(1 + |\vec{r}|^2)$ is derived and verified against a table of five representative states ranging from pure ($|\vec{r}|=1$, purity 1) to maximally mixed ($|\vec{r}|=0$, purity 0.5).

**9.1 — Non-uniqueness of decompositions.** One of the most conceptually striking facts in quantum information is demonstrated: the 50/50 mixture of $|0\rangle$ and $|1\rangle$ (a $z$-axis mixture) and the 50/50 mixture of $|{+}\rangle$ and $|{-}\rangle$ (an $x$-axis mixture) produce identically the same density matrix $I/2$. No measurement can ever distinguish the two preparation procedures — the density matrix captures everything experimentally accessible.

**9.2 — Interactive mixture.** A slider $p$ moves between two selectable cardinal states. The Bloch vector slides along the straight chord connecting the two pure points, illustrating that classical mixing is geometrically linear — a convex combination.

### Section 10 — Decoherence: Dephasing

The dephasing channel is implemented in the `dephase` function and applied to $|{+}\rangle$ for six values of $\lambda$. The table output shows $r_x$ shrinking monotonically while $r_z$ stays at $0$, reaching $\vec{r} = 0$ at $\lambda = \frac{1}{2}$. An interactive widget and an animation (Section 10.1) then let the student choose any initial pure state and watch the Bloch vector retract toward the $z$-axis as $\lambda$ increases — a visceral demonstration of the destruction of quantum coherence.

### Sections 11 & 12 — Cheat Sheet and Exercises

A concise reference table collects every key formula from the notebook. Ten exercises span immediate verification tasks (compute expectation values for a specific state), deeper algebraic checks (show purity is unitarily invariant), implementation challenges (amplitude-damping channel), and a bonus preview of the entanglement content in the next notebook (compute the reduced density matrix of one qubit in a Bell pair).

---

## Key Takeaways

- A single qubit has exactly **two real degrees of freedom** after normalisation and global-phase removal: the colatitude $\theta$ and azimuth $\varphi$ on a sphere.
- The factor of $\theta/2$ in the Bloch parameterisation is not a convention — it reflects the **spinor nature** of the qubit and implies that $4\pi$ rotation (not $2\pi$) is needed to return the quantum state to itself.
- Every **single-qubit gate** is a rigid rotation of the Bloch sphere; this geometric picture makes it immediate which states are affected and which are not (states on the rotation axis are fixed points).
- **Orthogonal states are antipodal**: states that are $90°$ apart in Hilbert space are $180°$ apart on the sphere — a direct consequence of the half-angle and one of the most useful facts for building quantum intuition.
- The **density matrix** $\rho = \frac{1}{2}(I + \vec{r}\cdot\vec{\sigma})$ generalises the Bloch vector to mixed states; the Bloch ball (sphere plus interior) is the complete space of all physically allowed single-qubit states.
- The same density matrix can arise from **infinitely many different classical mixtures** of pure states — only the Bloch vector is experimentally accessible, not the specific decomposition used to prepare it.
- **Dephasing** shrinks the equatorial components of the Bloch vector toward zero while preserving the $z$-component, providing the simplest concrete model of how environmental noise destroys quantum coherence.

---

## Further Reading & Citations

1. **Nielsen, M. A., & Chuang, I. L.** (2010). *Quantum Computation and Quantum Information* (10th anniversary ed.). Cambridge University Press.
   — Chapter 1 (§1.2) introduces the qubit and Bloch sphere; Chapter 2 covers the density operator formalism comprehensively. The standard reference for the field.

2. **Preskill, J.** (1998). *Lecture Notes for Physics 229: Quantum Information and Computation*. California Institute of Technology.
   — Chapter 2 gives a thorough, graduate-level treatment of the Bloch sphere, density matrices, and quantum operations. Freely available at [http://theory.caltech.edu/~preskill/ph229/](http://theory.caltech.edu/~preskill/ph229/).

3. **Sakurai, J. J., & Napolitano, J.** (2017). *Modern Quantum Mechanics* (3rd ed.). Cambridge University Press.
   — Chapter 1 develops spin-$\frac{1}{2}$ systems from the Stern–Gerlach experiment; Chapter 3 covers rotation operators and spinors. The physics perspective that motivates the half-angle.

4. **Wilde, M. M.** (2017). *Quantum Information Theory* (2nd ed.). Cambridge University Press.
   — Part II treats quantum channels, the Kraus representation, and decoherence models (including dephasing and amplitude damping) with mathematical rigour. Also available on arXiv: [arXiv:1106.1445](https://arxiv.org/abs/1106.1445).

5. **Bloch, F.** (1946). Nuclear induction. *Physical Review*, 70(7–8), 460–474.
   — The original paper introducing what became known as the Bloch sphere, in the context of nuclear magnetic resonance — a beautiful example of the same mathematics arising independently in physics and quantum information.

6. **Bengtsson, I., & Zyczkowski, K.** (2006). *Geometry of Quantum States: An Introduction to Quantum Entanglement*. Cambridge University Press.
   — Chapters 1–5 place the Bloch ball in the broader mathematical context of convex sets, quantum state space geometry, and measures of mixedness. For the geometrically-minded reader.

7. **Gyongyosi, L., Imre, S., & Nguyen, H. V.** (2018). A survey on quantum channel capacities. *IEEE Communications Surveys & Tutorials*, 20(2), 1149–1205. [arXiv:1801.02019](https://arxiv.org/abs/1801.02019).
   — Places the dephasing and amplitude-damping channels introduced in this notebook within the broader context of quantum error and channel capacity theory.

---

<!-- 
SEO TAGS / KEYWORDS:
bloch sphere, density matrix, qubit, quantum computing, quantum mechanics,
pauli matrices, quantum gates, unitary evolution, rotation operator,
spinor, spin half, quantum superposition, quantum state, mixed state,
pure state, quantum decoherence, dephasing channel, quantum noise,
purity measure, quantum information theory, Stern-Gerlach experiment,
quantum measurement, Born rule, quantum visualization, ipywidgets,
matplotlib 3d, quantum pedagogy, FDP quantum computing, quantum entanglement,
qubit parameterization, bloch vector, quantum bloch ball, quantum coherence,
bloch sphere animation, quantum gates visualization, hadamard gate,
quantum foundations, quantum hardware noise, reduced density matrix,
amplitude damping, quantum channel, numpy quantum simulation
-->

---

## Related Notebooks in This Series

| # | Notebook | Topics |
|---|---|---|
| 1–2 | [Demo1-2\_Double\_Slit\_and\_Stern\_Gerlach\_FDP.ipynb](Demo1-2_Double_Slit_and_Stern_Gerlach_FDP.ipynb) | Double-slit interference, Stern–Gerlach experiment, probability amplitudes |
| 3 | [Demo3\_QMPostulates\_BraKet\_Bloch.ipynb](Demo3_QMPostulates_BraKet_Bloch.ipynb) | Five postulates of QM, Dirac notation, Pauli algebra |
| **4** | **[Demo4\_BlochSphere\_DensityMatrix.ipynb](Demo4_BlochSphere_DensityMatrix.ipynb)** | **Bloch sphere, density matrix, dephasing — this notebook** |
| 5 | [Demo5\_Purity\_Coherence\_Entanglement.ipynb](Demo5_Purity_Coherence_Entanglement.ipynb) | Purity, coherence measures, two-qubit entanglement |
| 6 | [Demo6\_Noise\_and\_Information\_Measures.ipynb](Demo6_Noise_and_Information_Measures.ipynb) | Quantum channels, entropy, information-theoretic measures |
| 7 | [Demo7\_Quantum\_Gates\_Demo.ipynb](Demo7_Quantum_Gates_Demo.ipynb) | Universal gate sets, circuit construction |
| 8 | [Demo8\_QuantumCircuits\_EntanglingGates\_WHT.ipynb](Demo8_QuantumCircuits_EntanglingGates_WHT.ipynb) | Entangling gates, Bell states, Walsh–Hadamard transform |
| 9 | [Demo9\_Qiskit\_Introduction.ipynb](Demo9_Qiskit_Introduction.ipynb) | Introduction to Qiskit, running circuits on simulators |
| 10 | [Demo10\_PennyLane\_Introduction\_Hands\_On.ipynb](Demo10_PennyLane_Introduction_Hands_On.ipynb) | PennyLane for variational quantum algorithms |
| 11 | [Demo11\_Qiskit\_PennyLane\_Synthesis\_and\_Comparison.ipynb](Demo11_Qiskit_PennyLane_Synthesis_and_Comparison.ipynb) | Comparing Qiskit and PennyLane workflows |
| 12 | [Demo12b\_Qiskit\_Oracles\_Primitives\_DJ.ipynb](Demo12b_Qiskit_Oracles_Primitives_DJ.ipynb) | Oracles, Deutsch–Jozsa algorithm |
| 13 | [Demo13b\_Bernstein\_Vazirani\_Qiskit.ipynb](Demo13b_Bernstein_Vazirani_Qiskit.ipynb) | Bernstein–Vazirani algorithm |
| 14 | [Demo14\_Simons\_Algorithm\_Qiskit.ipynb](Demo14_Simons_Algorithm_Qiskit.ipynb) | Simon's algorithm |
| 15 | [Demo15\_Grover\_Qiskit\_FDP.ipynb](Demo15_Grover_Qiskit_FDP.ipynb) | Grover's search algorithm |

---

*Prepared for the Faculty Development Programme (FDP) on Quantum Computing, June–July 2026.*
*All code requires only NumPy, Matplotlib, and ipywidgets — no quantum hardware access needed.*
