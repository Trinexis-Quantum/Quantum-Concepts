# ⚛️ Hands-on 1 & 2 — The Double-Slit & Stern–Gerlach Experiments: Gateway to the Qubit

[![Python 3.9+](https://img.shields.io/badge/python-3.9%2B-blue?logo=python&logoColor=white)](https://www.python.org/)
[![Jupyter Notebook](https://img.shields.io/badge/Jupyter-Notebook-orange?logo=jupyter&logoColor=white)](https://jupyter.org/)
[![NumPy](https://img.shields.io/badge/NumPy-2.x-013243?logo=numpy&logoColor=white)](https://numpy.org/)
[![Matplotlib](https://img.shields.io/badge/Matplotlib-3.x-11557c)](https://matplotlib.org/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Series](https://img.shields.io/badge/Series-Quantum_Computing_Education-6929C4?style=flat-square)](https://github.com/Trinexis-Quantum/Quantum-Concepts)

---

## Overview

Imagine throwing pebbles through two gaps in a fence. Each pebble clearly passes through one gap or the other, and the pattern of pebbles on the far wall is simply the sum of two independent piles — nothing surprising there. Now replace the pebbles with electrons. Something astonishing happens: even though each electron arrives as a single, indivisible click on the detector (exactly like a pebble), the *pattern* built up by many such clicks is not two piles but a ripple of bright and dark **interference fringes** — the same pattern you see when water waves pass through two gaps simultaneously. This is the double-slit experiment, and it is the single most important experiment in quantum mechanics. It tells us that quantum objects are not described by probabilities, but by **complex-valued amplitudes** whose squares give probabilities (the Born rule), and that indistinguishable paths add as amplitudes rather than as probabilities.

This notebook takes that insight one step further with the **Stern–Gerlach experiment**, where a beam of silver atoms is split by a magnetic field into exactly **two** discrete spots — never a continuous smear, regardless of how the atoms were prepared. This two-outcome structure is not merely a curiosity: it is the birth certificate of the **qubit**, the fundamental unit of quantum information. Where the double slit gives us the *mathematics* of superposition (complex amplitudes, the Born rule, the interference term), the Stern–Gerlach experiment gives us the *geometry* of a two-level quantum system: the Bloch sphere, incompatible observables, and the physical meaning of measurement collapse.

By the end of this notebook, the two threads are woven together in **Part III**, where single-qubit gates and a miniature simulation of Deutsch's algorithm demonstrate that a quantum computer is, at its heart, a carefully engineered double-slit experiment — one in which we use controlled interference to amplify the probability of correct answers and suppress the probability of wrong ones. No prior quantum mechanics is assumed. All simulations are built from scratch using only NumPy and Matplotlib, so every formula is visible, every number is computable, and every concept has a picture to go with it.

---

## Learning Objectives

By the end of this notebook you will be able to:

- **Explain the Born rule** — why quantum probabilities are squared magnitudes of complex amplitudes — and connect it to the observed double-slit fringe pattern.
- **Distinguish classical (incoherent) probability addition from quantum (coherent) amplitude addition**, and state precisely when each applies.
- **Explain why which-path information destroys interference**, and connect this to the concept of decoherence in quantum computers.
- **Decompose the full quantum probability** $P_{12}$ into its classical part $(P_1 + P_2)$ and its quantum interference term $2\,\mathrm{Re}(\phi_1^*\phi_2)$.
- **Represent a qubit state as a 2-component complex vector** and compute Born-rule probabilities for any measurement direction.
- **Describe the Stern–Gerlach experiment** and explain why it is the physical realisation of a qubit measurement.
- **Interpret the Bloch sphere geometrically**: identify where $|0\rangle$, $|1\rangle$, $|{\pm}x\rangle$, and $|{\pm}y\rangle$ live, and relate Stern–Gerlach axes to Bloch-sphere axes.
- **Apply the single-qubit gates X, Z, and H** by hand and in simulation, and verify their key properties (self-inverse H, phase-flip Z, bit-flip X).
- **Explain why measuring along the $x$-axis is equivalent to applying a Hadamard gate then measuring along $z$**.
- **Trace through Deutsch's algorithm** step by step and articulate why one oracle query suffices to distinguish a constant from a balanced function.

---

## Background & Theory

### The Quantum Rule That Changes Everything

Classical physics describes uncertainty with ordinary probabilities: if there are two paths to an outcome, their probabilities just add. Quantum mechanics replaces probabilities with **probability amplitudes** — complex numbers $\phi$ — and the rule changes dramatically:

$$P = |\phi|^2 \quad \text{(Born rule)}$$

When two indistinguishable paths lead to the same outcome, the **amplitudes** add before squaring:

$$P_{12} = |\phi_1 + \phi_2|^2 = |\phi_1|^2 + |\phi_2|^2 + 2\,\mathrm{Re}(\phi_1^*\phi_2)$$

That last term — the **interference term** — has no classical counterpart. It can be positive (constructive interference, bright fringes) or negative (destructive interference, dark fringes) depending on the relative **phase** between $\phi_1$ and $\phi_2$.

### The Double-Slit Experiment

In the double-slit setup, a quantum particle (say, an electron) can travel through either of two slits to reach a point $x$ on a distant screen. If we do not — even in principle — record which slit the electron used, the two paths are **indistinguishable** and we must add amplitudes:

$$\phi(x) = \phi_1(x) + \phi_2(x), \qquad P(x) = |\phi_1(x) + \phi_2(x)|^2$$

In the far-field (Fraunhofer) approximation, the amplitude from each slit carries a phase proportional to the path-length difference $\delta = kd\,x/L$ (where $d$ is the slit separation, $L$ the screen distance, and $k = 2\pi/\lambda$ the wave number). This produces the familiar fringe pattern:

$$I_{12}(x) = I_1 + I_2 + 2\sqrt{I_1 I_2}\,\cos\delta$$

The moment we *can* tell which slit was used — even if we never look at the record — the paths become distinguishable and interference vanishes:

$$P_{12} = P_1 + P_2 \quad \text{(no interference, which-path known)}$$

This is not a matter of experimental clumsiness. It is a fundamental feature of quantum mechanics. The same mechanism, at the scale of a quantum processor, is called **decoherence** and is the primary engineering challenge in building quantum computers.

### The Stern–Gerlach Experiment and the Qubit

In 1922, Walther Gerlach and Otto Stern sent silver atoms through an inhomogeneous magnetic field. Classical physics predicted a continuous smear of deflections (because the atomic magnetic dipole can point in any direction). What they found was a clean split into **exactly two** discrete beams. This quantization of angular momentum — spin-$\tfrac{1}{2}$ — is nature's own two-level system.

We label the two outcomes **spin-up** $|{+}z\rangle \equiv |0\rangle$ and **spin-down** $|{-}z\rangle \equiv |1\rangle$. An arbitrary qubit state is a superposition:

$$|\psi\rangle = \alpha\,|0\rangle + \beta\,|1\rangle, \qquad |\alpha|^2 + |\beta|^2 = 1, \quad \alpha,\beta\in\mathbb{C}$$

The **Born rule** gives the measurement probabilities: $P(0) = |\alpha|^2$, $P(1) = |\beta|^2$.

If we measure not along $z$ but along an axis tilted by angle $\theta$ from $z$, the probability of the "up" outcome is:

$$P\!\left(+\hat{n}_\theta\right) = \cos^2\!\!\left(\frac{\theta}{2}\right)$$

This is the spin analogue of Malus's Law in optics and is the precise mathematical meaning of "the state is a superposition."

### The Bloch Sphere

Every normalised qubit state (up to a global phase) can be written as:

$$|\psi\rangle = \cos\!\frac{\theta}{2}\,|0\rangle + e^{i\varphi}\sin\!\frac{\theta}{2}\,|1\rangle$$

for angles $\theta \in [0, \pi]$ and $\varphi \in [0, 2\pi)$. This maps every qubit state to a unique point on the surface of the **unit sphere** — the **Bloch sphere**. The north pole is $|0\rangle$, the south pole is $|1\rangle$, and the equatorial states are equal-weight superpositions. Crucially, the three Stern–Gerlach measurement axes ($z$, $x$, $y$) correspond exactly to the three Cartesian axes of this sphere. Rotating the qubit state on the Bloch sphere is exactly what a quantum gate does.

### Single-Qubit Gates

Allowed operations on a qubit are **unitary** $2\times 2$ matrices — geometrically, rotations of the Bloch sphere. The three key gates in this notebook are:

| Gate | Matrix | Action |
|------|--------|--------|
| $X$ (NOT) | $\begin{pmatrix}0&1\\1&0\end{pmatrix}$ | Bit-flip: $|0\rangle \leftrightarrow |1\rangle$ (180° rotation about $x$) |
| $Z$ (Phase flip) | $\begin{pmatrix}1&0\\0&-1\end{pmatrix}$ | Phase-flip: $|{+}x\rangle \leftrightarrow |{-}x\rangle$ (180° rotation about $z$) |
| $H$ (Hadamard) | $\frac{1}{\sqrt{2}}\begin{pmatrix}1&1\\1&-1\end{pmatrix}$ | $|0\rangle \to |{+}x\rangle$, $|1\rangle \to |{-}x\rangle$ (swaps $x$- and $z$-axes) |

The Hadamard gate is the discrete counterpart of the double slit: it puts a single computational basis state into an equal superposition of both basis states, setting up the conditions for interference.

### Deutsch's Algorithm: Interference Solves a Problem

Given a black-box function $f:\{0,1\}\to\{0,1\}$, is $f$ **constant** (both outputs equal) or **balanced** (outputs differ)? Classically you must call $f$ twice. Deutsch's 1985 algorithm decides in **one** call, using the circuit $H \to U_f \to H$, where $U_f$ encodes $f$ as a phase: $U_f|x\rangle = (-1)^{f(x)}|x\rangle$. The final measurement lands on $|0\rangle$ with certainty if $f$ is constant, and on $|1\rangle$ with certainty if $f$ is balanced. The mechanism is exactly the qubit interferometer of Section 3.3: constructive interference onto the correct answer, destructive interference away from the wrong one.

---

## Prerequisites

**Mathematical background:**
- Complex numbers: modulus, argument, Euler's formula $e^{i\theta} = \cos\theta + i\sin\theta$
- Linear algebra: vectors, matrix multiplication, inner products
- Basic probability: probability distributions, expectation values

**Programming background:**
- Python at an introductory level (loops, functions, NumPy arrays)
- Familiarity with Jupyter notebooks (running cells, reading output)

**Physics background:**
- High-school wave physics is helpful (wavelength, frequency, interference) but not required — the notebook builds everything from scratch
- No prior quantum mechanics is assumed

**Software environment:**
- Python 3.9 or later
- `numpy >= 1.24`, `matplotlib >= 3.7`
- A Jupyter environment (JupyterLab, Jupyter Notebook, VS Code with the Jupyter extension, or Google Colab)

---

## Notebook Walkthrough

### Setup Cell

The very first code cell imports NumPy and Matplotlib, sets a random seed (`42`) for reproducibility, and configures global plot aesthetics. **Always run this cell first** — every subsequent cell depends on it. The seed is important: it ensures that the Monte-Carlo simulations produce the same figures every time, making the notebook suitable for classroom demonstrations.

---

### Part I — The Double-Slit Experiment

#### Section 1.1 — Bullets: Classical Particles (Probabilities Add)

**Why this section exists:** Before introducing quantum weirdness, the notebook establishes what the classical expectation *should* look like, so that the quantum result later stands in sharp relief.

The simulation fires 100,000 classical "bullets" through a two-slit geometry. Each bullet passes through hole 1 or hole 2 with equal probability, then scatters with a Gaussian spread. The key check is that the combined distribution $P_{12}$ is *exactly* the sum $P_1 + P_2$ (the printed maximum difference is 0). This is the baseline: classical probability addition with zero interference. If quantum particles behaved this way, quantum computing would be impossible.

#### Section 1.2 — Waves: Amplitudes Add, Then Square

**Why this section exists:** To show that interference is a natural consequence of the "add-then-square" rule for complex amplitudes.

Using the far-field (Fraunhofer) approximation, the notebook computes complex amplitudes $\phi_1$ and $\phi_2$ for each slit, each carrying a phase proportional to the path-length difference $\delta$. The intensity $I_{12} = |\phi_1 + \phi_2|^2$ produces the classic fringe pattern, and the simulation confirms that the central fringe is exactly *twice* the classical sum ($I_{12}(0) = 4$ vs. $I_1 + I_2 = 2$). The diffraction envelope (single-slit sinc² factor) modulates the fringes realistically.

#### Section 1.3 — Electrons: Discrete Lumps, Wave Statistics

**Why this section exists:** To resolve the apparent paradox — electrons arrive as discrete clicks (like bullets) but accumulate into a wave pattern (like water waves).

The simulation treats $|\phi_1 + \phi_2|^2$ as a probability mass function and samples individual "click" events one at a time. Four panels show 50, 500, 5,000, and 50,000 electrons. With 50 electrons the pattern looks random; with 50,000 the interference fringes are unmistakable. The message: **each electron is a particle; only the statistics of many electrons reveal the wave**. This is exactly what Feynman called "the most mysterious fact in quantum mechanics."

#### Section 1.4 — Watching: Which-Path Information Destroys Interference

**Why this section exists:** This is the conceptual heart of the first half — and the most important lesson for understanding decoherence in quantum computers.

Two runs of 50,000 electrons are compared side by side. The "unwatched" run samples from the interference law; the "watched" run assigns each electron definitely to hole 1 or hole 2 and samples from the corresponding single-slit distribution. The fringes completely disappear. Critically, this is not because the detector physically disturbs the electrons — it is because the two paths have become *distinguishable in principle*. Once distinguishable, $|\phi_1 + \phi_2|^2$ collapses to $|\phi_1|^2 + |\phi_2|^2$, and interference is gone.

#### Section 1.5 — The Interference Term, Made Explicit

**Why this section exists:** To give students a formula they can point to and say "this is what a quantum computer exploits."

The identity $P_{12} = (P_1 + P_2) + 2\,\mathrm{Re}(\phi_1^*\phi_2)$ is plotted in three components: the classical part (gray), the oscillating interference term (green), and the total (red). An `assert` statement confirms the identity holds to floating-point precision — no approximations. Students can see exactly where constructive and destructive interference occur and what the "quantum contribution" looks like graphically.

#### Section 1.6 — Summary of Part I

A concise conceptual summary closes Part I, noting that the electron's continuous position is impractical for computing (infinitely many outcomes). The solution — a quantum system with exactly *two* outcomes — motivates Part II.

---

### Part II — The Stern–Gerlach Experiment → the Qubit

#### Section 2.1 — The Qubit as a Complex Vector

**Why this section exists:** To introduce the mathematical structure of a qubit before the physics of the Stern–Gerlach experiment, so students have the notation in hand.

The six standard states ($|0\rangle$, $|1\rangle$, $|{\pm}x\rangle$, $|{\pm}y\rangle$) are defined as NumPy arrays. Note the imaginary unit $i$ in $|{+}y\rangle = \tfrac{1}{\sqrt{2}}(|0\rangle + i|1\rangle)$ — the same complex arithmetic that Feynman insisted on in Part I now appears in distinguishing the $y$-measurement from the $x$-measurement. The Pauli matrices $X$, $Y$, $Z$ and the Hadamard $H$ are also defined here for use throughout the rest of the notebook.

#### Section 2.2 — A Single SG Box: Quantization and the Born Rule

**Why this section exists:** To simulate the Stern–Gerlach experiment directly and confirm that measurement always yields exactly two outcomes.

Four helper functions are introduced: `unit` (normalise a direction vector), `axis_operator` (build the spin operator $\hat{n}\cdot\boldsymbol{\sigma}$ for direction $\hat{n}$), `plus_eigenstate` (find the spin-up eigenstate along $\hat{n}$), and `measure_axis` (perform one projective measurement, returning the outcome $\pm 1$ and the post-measurement collapsed state). Two scatter plots then show the "screen": a pure $|0\rangle$ beam measured along $z$ lands entirely at the upper spot; an unpolarised beam (random states from the full Bloch sphere) splits 50/50 into two distinct spots. **Two spots, always — never a smear.** This is quantization.

#### Section 2.3 — Sequential SG: Superposition, Incompatibility, Erasure

**Why this section exists:** Three cascades in one simulation reveal all the key structural features of qubit measurement.

- **(A) z → z**: A $|0\rangle$ beam measured along $z$, then measured along $z$ again, yields 100% up both times. Measurement is **repeatable**: the state collapses and stays collapsed.
- **(B) z → x**: A $|0\rangle$ beam (definite along $z$) measured along $x$ yields exactly 50% up and 50% down. Being definite along $z$ means being maximally indefinite along $x$. These are **incompatible observables**.
- **(C) z → x → z**: Selecting the $+x$ branch and measuring along $z$ again gives 50/50 — the original $z$-information has been **erased** by the intervening $x$-measurement. This is measurement collapse in action, and it is the discrete spin analogue of which-path detection from Part I.

#### Section 2.4 — The Measurement Law: $P(+) = \cos^2(\theta/2)$

**Why this section exists:** To give students the precise quantitative formula for Born-rule probabilities and let them verify it against simulation.

A $|0\rangle$ state is measured along axes tilted from $z$ by angles $0°$ to $180°$, with 4,000 Monte-Carlo trials per angle. The empirical points lie on the theoretical curve $\cos^2(\theta/2)$ within statistical noise. The three landmark values are intuitive: $\theta=0°$ (same axis, $P=1$), $\theta=90°$ (the SG z→x split, $P=0.5$), and $\theta=180°$ (opposite pole, $P=0$).

#### Section 2.5 — The Bloch Sphere

**Why this section exists:** To give students the geometric picture that unifies everything in Part II.

A 3D wireframe sphere is plotted with six labelled state vectors: $|0\rangle$ and $|1\rangle$ at the $z$-poles, $|{\pm}x\rangle$ on the $x$-axis, and $|{\pm}y\rangle$ on the $y$-axis. The axis labels read "X (SG-x)", "Y (SG-y)", "Z (SG-z)" — making explicit that the SG apparatus direction is literally a Bloch-sphere axis. Students can now read any Stern–Gerlach result off the sphere geometrically.

#### Section 2.6 — Summary of Part II

Collects the four concepts the qubit inherits from the Stern–Gerlach experiment: two-level quantization, the Born rule for tilted measurements, incompatible observables and collapse, and the Bloch sphere. Notes that rotations of the Bloch sphere are quantum gates — setting up Part III.

---

### Part III — Bridge to Quantum Computing

#### Section 3.1 — Single-Qubit Gates X, Z, H

**Why this section exists:** To make gates concrete before using them algorithmically.

The three key gates are verified numerically: $X|0\rangle = |1\rangle$, $X|1\rangle = |0\rangle$, $H|0\rangle = |{+}x\rangle$, $H|1\rangle = |{-}x\rangle$, $Z|{+}x\rangle = |{-}x\rangle$, $H^2 = I$. Two additional checks confirm that $Z$ is a phase flip (invisible in the $z$-basis but physically meaningful when you measure in the $x$-basis) and that $H$ is its own inverse. These are not just abstract matrix facts — each one has a Bloch-sphere rotation interpretation.

#### Section 3.2 — The Hadamard IS the SG z→x Experiment

**Why this section exists:** To create a memorable conceptual bridge between the physical experiment and the gate formalism.

For four different input states, the notebook computes two quantities: (1) the probability of the $+x$ outcome measured directly, and (2) the probability of the $|0\rangle$ outcome after applying $H$ then measuring along $z$. They match exactly for every state tested. Changing measurement basis by applying $H$ and measuring along the standard $z$-axis is identically the same operation as rotating the physical SG magnet to the $x$-axis. The two descriptions are one.

#### Section 3.3 — The Qubit Interferometer: A Discrete Double Slit

**Why this section exists:** To close the loop between Part I and Part III — the qubit interferometer is the double-slit experiment in miniature.

The circuit $|0\rangle \xrightarrow{H}$ (superposition) $\xrightarrow{P_\varphi}$ (relative phase) $\xrightarrow{H}$ (recombine) is simulated over a full $2\pi$ sweep of phases. The probability $P(0) = \cos^2(\varphi/2)$ traces a perfect interference fringe — the same formula as Part I's two-slit pattern, with the relative phase $\varphi$ replacing the path-difference phase $\delta$. This circuit is the seed from which most quantum algorithms grow.

#### Section 3.4 — Deutsch's Algorithm: Interference Solves a Problem

**Why this section exists:** To show that the interference exploited in Sections 1–3.3 is computationally useful — a genuine quantum speedup, even if only exponentially small for one bit.

The oracle $U_f$ is implemented as a diagonal matrix $\mathrm{diag}((-1)^{f(0)},\,(-1)^{f(1)})$ (the "phase-kickback" form standard in quantum computing texts). The circuit $H \to U_f \to H$ is applied to $|0\rangle$, and the final measurement determines constant vs. balanced in one step. All four Boolean functions are tested and correctly classified. The closing print statement makes the lesson explicit: interference reads a global property of $f$ in a single query, which is quantum advantage in its simplest possible form.

#### Section 3.5 — The Big Picture

A summary table maps each experiment to the quantum-computing concept it introduces, and the capstone takeaway is stated: quantum computation is the art of engineering amplitudes so that wrong answers interfere destructively and right answers interfere constructively — the double slit, scaled up.

---

## Key Takeaways

- **The Born rule is the foundation of everything.** Probabilities are squared magnitudes of complex amplitudes ($P = |\phi|^2$), not the amplitudes themselves. This single fact is responsible for interference, superposition, and every quantum speedup.
- **Indistinguishability is the switch that turns interference on.** When two paths to the same outcome cannot be told apart, amplitudes add and interference appears. The moment a record of "which path" exists — even in principle, even unread — probabilities add instead and interference vanishes. This is not a measurement disturbance; it is a fundamental change in what the laws of physics say.
- **Decoherence is which-path information, at scale.** Every uncontrolled interaction between a qubit and its environment is a which-path detector in disguise. Protecting quantum coherence is therefore the central engineering challenge of quantum computing hardware.
- **The Stern–Gerlach experiment is the qubit.** Nature offers a two-level system with discrete, born-rule probabilities, an incompatibility structure between measurement axes, and a clean geometric picture (the Bloch sphere). This is not an analogy for a qubit — it literally is a qubit.
- **The Hadamard gate and a 90° Stern–Gerlach rotation are the same operation.** Quantum gates are not abstract symbols; they correspond to real physical transformations. The Hadamard creates superposition by rotating the Bloch sphere, exactly as tilting the SG magnet by 90° creates a 50/50 split.
- **Every quantum algorithm is an interferometer.** The H–phase–H circuit of Section 3.3 already contains the essential structure of Deutsch's algorithm, the Bernstein–Vazirani algorithm, quantum phase estimation, and Grover's search: prepare a superposition, apply a phase encoding some information, then recombine and read off the result.
- **Quantum advantage is not magic — it is engineered interference.** A quantum computer does not "try all answers at once"; it arranges amplitude cancellations so that wrong answers are suppressed and the correct answer is amplified. Understanding this deeply is the key to reading, designing, and criticising quantum algorithms.

---

## Further Reading & Citations

1. **Feynman, R. P., Leighton, R. B., & Sands, M. (1965).** *The Feynman Lectures on Physics, Volume III: Quantum Mechanics.* Addison-Wesley. — Chapter 1 ("Quantum Behavior") is the direct inspiration for Part I of this notebook; Chapters 5–6 cover spin-½ and the Stern–Gerlach experiment. Freely available at [https://www.feynmanlectures.caltech.edu/](https://www.feynmanlectures.caltech.edu/).

2. **Gerlach, W., & Stern, O. (1922).** Der experimentelle Nachweis der Richtungsquantelung im Magnetfeld. *Zeitschrift für Physik*, **9**(1), 349–352. [https://doi.org/10.1007/BF01326983](https://doi.org/10.1007/BF01326983) — The original Stern–Gerlach paper. A landmark in 20th-century physics; readable in translation with modern commentary.

3. **Deutsch, D. (1985).** Quantum Theory, the Church–Turing Principle and the Universal Quantum Computer. *Proceedings of the Royal Society of London. Series A*, **400**(1818), 97–117. [https://doi.org/10.1098/rspa.1985.0070](https://doi.org/10.1098/rspa.1985.0070) — The paper that introduced Deutsch's algorithm and the concept of a universal quantum computer. Sections 3.4–3.5 of this notebook implement its simplest instance.

4. **Nielsen, M. A., & Chuang, I. L. (2010).** *Quantum Computation and Quantum Information* (10th Anniversary Edition). Cambridge University Press. — The field's canonical reference. Chapters 1–4 cover the qubit model, the Bloch sphere, single-qubit gates, and Deutsch's algorithm at the level assumed by the later notebooks in this series.

5. **Sakurai, J. J., & Napolitano, J. (2017).** *Modern Quantum Mechanics* (2nd ed.). Cambridge University Press. — Chapter 1 develops the Stern–Gerlach experiment as the foundation of quantum mechanics and introduces the Dirac bra-ket formalism used throughout this notebook. The clearest graduate-level treatment of spin-½.

6. **Griffiths, D. J., & Schroeter, D. F. (2018).** *Introduction to Quantum Mechanics* (3rd ed.). Cambridge University Press. — Chapters 1 and 4 provide the wavefunction picture (Part I) and the spin-½ formalism (Part II) at an accessible undergraduate level, with numerous worked examples.

7. **Preskill, J. (2018).** Quantum Computing in the NISQ Era and Beyond. *Quantum*, **2**, 79. [https://doi.org/10.22331/q-2018-08-06-79](https://doi.org/10.22331/q-2018-08-06-79) — An accessible overview of why near-term quantum devices face decoherence as their primary obstacle — the concept whose physical origin this notebook establishes.

---

<!-- 
SEO TAGS / KEYWORDS
quantum computing tutorial, double slit experiment, Stern-Gerlach experiment, qubit, quantum superposition, probability amplitude, Born rule, wave-particle duality, quantum interference, which-path information, decoherence, Bloch sphere, Hadamard gate, Pauli matrices, single qubit gates, quantum measurement, wavefunction collapse, Feynman lectures, Deutsch algorithm, quantum advantage, quantum computing education, quantum computing education series by Trinexis, Quantum Computing Education Series by Trinexis, hands-on quantum, quantum simulation Python, NumPy quantum, quantum amplitude, complex amplitude, interference fringe, incompatible observables, spin half, spin measurement, quantum probability, quantum algorithms introductory, quantum computing beginner
-->

---

## Related Notebooks in This Series

| # | Notebook | Topic |
|---|----------|-------|
| **1–2** | `Demo1-2_Double_Slit_and_Stern_Gerlach.ipynb` | **You are here** — Double slit, Stern–Gerlach, qubit foundations |
| 3 | `Demo3_QMPostulates_BraKet_Bloch.ipynb` | Quantum mechanics postulates, Dirac notation, Bloch sphere (deeper) |
| 4 | `Demo4_BlochSphere_DensityMatrix.ipynb` | Density matrices, mixed states, Bloch sphere for mixed states |
| 5 | `Demo5_Purity_Coherence_Entanglement.ipynb` | Purity, coherence measures, entanglement |
| 6 | `Demo6_Noise_and_Information_Measures.ipynb` | Quantum noise channels, entropy, quantum information measures |
| 7 | `Demo7_Quantum_Gates_Demo.ipynb` | Universal gate sets, multi-qubit gates |
| 8 | `Demo8_QuantumCircuits_EntanglingGates_WHT.ipynb` | Quantum circuits, entangling gates, Walsh–Hadamard transform |
| 9 | `Demo9_Qiskit_Introduction.ipynb` | Introduction to Qiskit |
| 10 | `Demo10_PennyLane_Introduction_Hands_On.ipynb` | Introduction to PennyLane |
| 11 | `Demo11_Qiskit_PennyLane_Synthesis_and_Comparison.ipynb` | Qiskit vs. PennyLane comparison |
| 12 | `Demo12b_Qiskit_Oracles_Primitives_DJ.ipynb` | Oracles and the Deutsch–Jozsa algorithm in Qiskit |
| 13 | `Demo13b_Bernstein_Vazirani_Qiskit.ipynb` | Bernstein–Vazirani algorithm |
| 14 | `Demo14_Simons_Algorithm_Qiskit.ipynb` | Simon's algorithm |
| 15 | `Demo15_Grover_Qiskit.ipynb` | Grover's search algorithm |

---

*Quantum Computing Education Series by Trinexis — Quantum Computing, 2026*
*Notebook authored for the series series. Simulations built with NumPy 2.0 and Matplotlib; no quantum framework required for this notebook.*
