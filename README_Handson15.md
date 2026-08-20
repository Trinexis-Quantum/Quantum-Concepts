# 🔍 Grover's Search Algorithm, Hands-On 15

[![Qiskit](https://img.shields.io/badge/Qiskit-2.5.0-6929C4?logo=qiskit&logoColor=white)](https://qiskit.org)
[![Qiskit Aer](https://img.shields.io/badge/Qiskit--Aer-0.17.2-6929C4)](https://qiskit.org/ecosystem/aer/)
[![Python](https://img.shields.io/badge/Python-3.9%2B-3776AB?logo=python&logoColor=white)](https://www.python.org/)
[![Jupyter](https://img.shields.io/badge/Jupyter-Notebook-F37626?logo=jupyter&logoColor=white)](https://jupyter.org)
[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)
[![Series](https://img.shields.io/badge/Series-Quantum_Computing_Education-6929C4?style=flat-square)](https://github.com/Trinexis-Quantum/Quantum-Concepts)

> **Education Series in Quantum Computing · Trinexis · 2026**
> *Hands-On Session 15 of 15*

---

## Overview

Imagine you need to find a single misspelled word hidden somewhere in a book with a million pages, but you can only check one page at a time and the word appears on exactly one page. Classically, you might have to check half a million pages on average before finding it. Grover's algorithm, introduced by Lov Grover in 1996, lets a quantum computer find that page using only about a thousand checks, a *quadratic speedup* that sounds almost magical. This notebook is your guided laboratory for understanding exactly where that speedup comes from and how to build it yourself in Qiskit.

The central idea is **quantum interference**, not the popular-science notion of "trying all answers at once." A quantum computer places its register in an equal superposition of all possible inputs, but if you measured right there, you would get a random answer with no advantage whatsoever. The real work happens through two carefully designed operations. The first, called the **phase oracle**, marks the answer by flipping the sign of its amplitude (a kind of invisible tag). The second, called the **diffuser**, reflects every amplitude about their average, which causes the marked state to grow while the others shrink. Each round of oracle followed by diffuser rotates the quantum state a small angle closer to the answer. After approximately $\frac{\pi}{4}\sqrt{N}$ rounds, measurement gives the answer with near-certainty.

This notebook builds every component from scratch, no high-level black boxes, so the physics stays visible at every step. You will watch amplitudes shift in bar charts, trace the quantum state as a rotating arrow in a two-dimensional plane, observe the catastrophic effect of running too many iterations, and finally see the quadratic speedup plotted against the exponentially-hard classical alternative. Four graded exercises (★ to ★★★★) let you test and extend your understanding, culminating in a generalisation called **amplitude amplification** that underpins many of the most important quantum algorithms known today.

---

## Learning Objectives

After completing this notebook, you will be able to:

- **Explain** why Grover's speedup comes from constructive interference of amplitudes, not from parallel evaluation of a function, and articulate this clearly to a non-expert.
- **Construct** a phase oracle for any set of marked bit-strings by composing X gates and a multi-controlled-Z gate in Qiskit.
- **Construct** the Grover diffuser as a reflection about the uniform superposition state, and understand why a global-phase correction is needed for clean geometry.
- **Derive** the rotation angle $\theta = 2\arcsin\!\sqrt{M/N}$ per iteration and the optimal stopping time $k^* \approx \frac{\pi}{4}\sqrt{N/M}$ from the geometric picture.
- **Predict** the success probability after any number of iterations using the formula $P(k) = \sin^2\!\!\left(\frac{2k+1}{2}\theta\right)$ and verify the prediction against simulation.
- **Recognise** and explain the over-rotation phenomenon: why running more iterations than $k^*$ *decreases* the success probability, and why Grover oscillates rather than converges.
- **Simulate** Grover's algorithm on the Aer simulator and read shot-based histograms and exact statevector results.
- **Generalise** the algorithm to multiple marked states and connect the speedup formula to real query complexity.
- **Sketch** the amplitude amplification framework (Exercise 4) and describe how it extends Grover to arbitrary starting states.

---

## Background & Theory

### The Search Problem

We have a function $f : \{0,1\}^n \to \{0,1\}$ available as a black box (an *oracle*). Exactly $M$ of the $N = 2^n$ possible inputs satisfy $f(x) = 1$; we want to find any one of them. Because the function is structureless, a black box, no classical algorithm can do better than checking inputs one by one. On average, you need $\sim N/(2M)$ queries, i.e. $O(N)$ in the worst case. Grover's algorithm solves this with $O\!\left(\sqrt{N/M}\right)$ queries, a provably optimal quantum algorithm for this problem (Bennett et al., 1997).

### Amplitudes, Probabilities, and Interference

In quantum mechanics, a register of $n$ qubits is described by a *state vector*, a list of $N = 2^n$ complex numbers called *amplitudes*, one per basis state. The probability of measuring basis state $|x\rangle$ is the *squared magnitude* of its amplitude: $P(x) = |\alpha_x|^2$. This squaring relationship is everything: a small amplitude can be made large by interference, and a large amplitude can be destroyed. Constructive interference means amplitudes from different "paths" add up in magnitude; destructive interference means they cancel.

The Hadamard layer $H^{\otimes n}$ creates a **uniform superposition**

$$|s\rangle = \frac{1}{\sqrt{N}} \sum_{x=0}^{N-1} |x\rangle$$

in which every amplitude is $1/\sqrt{N}$. Measuring now gives a uniformly random answer, no advantage. The uniform superposition contains no computational information by itself. All the work is done in the two subsequent operations.

### The Phase Oracle

The oracle $U_f$ is a unitary that encodes $f$ as a *phase*:

$$U_f\,|x\rangle = (-1)^{f(x)}\,|x\rangle$$

This is equivalent to the operator $U_f = I, 2\sum_{x\,:\,f(x)=1} |x\rangle\langle x|$. Geometrically it is a **reflection** about the subspace of non-solutions: it leaves every non-solution unchanged and flips the sign of every solution. Crucially, the *probability* $|\alpha_x|^2$ is unchanged by a sign flip, so after one oracle call on the uniform superposition, the measurement distribution is still flat. The oracle has planted a phase difference that is *invisible to measurement alone* but exploitable by interference.

Practically, this is implemented by placing X gates on the qubits that should be 0 in the target bit-string (converting the target pattern to all-1s), applying a multi-controlled-Z gate (which phases $|1\cdots 1\rangle$ by $-1$), and restoring the X gates. The notebook function `phase_oracle(n, marked)` does exactly this.

### The Diffuser (Grover Diffusion Operator)

The diffuser is a **reflection about the uniform state** $|s\rangle$:

$$D = 2|s\rangle\langle s|, I$$

In coordinates, this replaces each amplitude $\alpha_x$ by $2\bar{\alpha}, \alpha_x$, where $\bar{\alpha}$ is the mean amplitude, hence the name *inversion about the mean*. A state that sits far below the mean (the solution's amplitude, which has gone negative after the oracle) gets reflected far above it. States sitting just above the mean (all non-solutions) are nudged downward. One application of oracle + diffuser therefore transfers amplitude from non-solutions to the solution.

The circuit realisation is $H^{\otimes n}$, then $X^{\otimes n}$, a multi-controlled-Z, $X^{\otimes n}$, $H^{\otimes n}$, which actually produces $-(2|s\rangle\langle s|, I)$. The notebook adds a global phase of $\pi$ to remove the minus sign, making the geometry exact.

### The Geometric Picture: Two Reflections = One Rotation

The entire $N$-dimensional Hilbert space is irrelevant to the dynamics. Grover's algorithm stays inside the two-dimensional plane spanned by:

$$|w\rangle = \frac{1}{\sqrt{M}} \sum_{x\,:\,f(x)=1} |x\rangle \quad \text{(the "winner" direction)}$$
$$|s'\rangle = \frac{1}{\sqrt{N-M}} \sum_{x\,:\,f(x)=0} |x\rangle \quad \text{(all non-solutions)}$$

The uniform starting state sits in this plane, tilted slightly toward $|w\rangle$:

$$|s\rangle = \sin\tfrac{\theta}{2}\,|w\rangle + \cos\tfrac{\theta}{2}\,|s'\rangle, \qquad \sin\tfrac{\theta}{2} = \sqrt{\tfrac{M}{N}}$$

The oracle is a reflection across the $|s'\rangle$ axis (it flips the $|w\rangle$ component). The diffuser is a reflection across the $|s\rangle$ line. A composition of two reflections whose mirror lines are separated by angle $\theta/2$ is a rotation by $\theta$. So **each Grover iteration rotates the state by $\theta$ toward $|w\rangle$**.

After $k$ iterations:

$$|\psi_k\rangle = \sin\!\left(\tfrac{2k+1}{2}\theta\right)|w\rangle + \cos\!\left(\tfrac{2k+1}{2}\theta\right)|s'\rangle$$
$$P_{\text{success}}(k) = \sin^2\!\!\left(\tfrac{2k+1}{2}\theta\right)$$

The state starts close to the $|s'\rangle$ axis (since $M \ll N$ means $\theta/2 \approx 0$) and rotates toward $|w\rangle$. We stop when the angle $\frac{2k+1}{2}\theta \approx \frac{\pi}{2}$, which gives:

$$k^* = \left\lfloor \frac{\pi/2}{\theta}, \frac{1}{2} \right\rceil \approx \frac{\pi}{4}\sqrt{\frac{N}{M}}$$

### The Quadratic Speedup

For $M = 1$, Grover needs $\approx \frac{\pi}{4}\sqrt{N}$ oracle calls versus $\approx N/2$ classical calls. The ratio grows as $\sqrt{N}$: for $n = 20$ qubits ($N \approx 10^6$), Grover needs roughly 800 calls where classical search needs 500,000. This speedup is **quadratic**, not exponential, and it is provably optimal: no quantum algorithm can search an unstructured database faster (Bennett et al., 1997).

### Over-Rotation: Why More Is Not Better

Because Grover is a rotation, not a convergent search, it overshoots if you apply too many iterations. The success probability $\sin^2(\frac{2k+1}{2}\theta)$ is a sinusoid in $k$; it peaks at $k^*$, then falls, rises again, and so on. Running $k^* + 1$ iterations can be as bad as running $0$ iterations. Stopping at the right moment requires knowing $M$ (or using randomised variants such as BBHT for unknown $M$).

### Amplitude Amplification

Grover's algorithm is the special case of the more general **amplitude amplification** primitive (Brassard et al., 2002). Any quantum algorithm that prepares a state $A|0\rangle$ can be used in place of the Hadamard layer, with the diffuser adapted to reflect about $A|0\rangle$ rather than $|s\rangle$. The rotation angle becomes $\theta = 2\arcsin|\langle w | A | 0\rangle|$, and the speedup applies whenever the initial overlap with the solution is small. Exercise 4 in the notebook constructs this explicitly.

---

## Prerequisites

### Knowledge

| Topic | Required Level |
|---|---|
| Complex numbers and linear algebra (vectors, inner products, matrices) | Comfortable |
| Quantum states, superposition, measurement in the computational basis | Familiar |
| Quantum gates: X, H, Z, multi-controlled gates | Familiar |
| Quantum circuits and the circuit model | Familiar |
| Python 3 and NumPy | Comfortable |

### Software

- **Python 3.9+** (Anaconda or Miniconda recommended)
- **Qiskit 2.5.0** and **Qiskit-Aer 0.17.2** (installed automatically in the setup cell if missing)
- **Jupyter Notebook** or **Google Colab** (the notebook is Colab-ready)
- `ipywidgets` (for the interactive explorer in Section 7)

### Recommended Prior Notebooks in This Series

- Demo7: Quantum Gates, X, Z, H, controlled gates
- Demo8: Quantum Circuits, Entangling Gates, Walsh-Hadamard Transform
- Demo9: Qiskit Introduction
- Demo12b: Qiskit Oracles and the Deutsch-Jozsa Algorithm (introduces the oracle/black-box model)
- Demo13b: Bernstein-Vazirani Algorithm (phase kickback and phase oracles)
- Demo14: Simon's Algorithm (hidden structure and quantum query complexity)

---

## Notebook Walkthrough

### Section 0, Setup

The first code cell performs a guarded install: it checks whether Qiskit is already present before invoking pip, making it safe to run in both Colab (where it installs) and a local environment (where it is a no-op). It then imports all libraries, fixes `np.random.seed(42)` for reproducibility across the whole course series, and sets a consistent matplotlib style. Four semantic colours are defined and reused throughout: red for marked/solution states, blue for unmarked states, black for the mean line, and green for optimal or "good" results. Keeping a consistent colour language across all plots is a pedagogical choice, once you learn what red means, you read every subsequent chart faster.

### Section 1, The Search Problem and Why Interference

This section establishes the problem (find an $x$ with $f(x) = 1$ given $f$ as a black box) and immediately addresses the most common misconception: that quantum speedup comes from evaluating $f$ on all inputs simultaneously. The text argues carefully that the uniform superposition $|s\rangle$, taken alone, contains no computational information, a measurement there would be uniformly random. The speedup is entirely due to what happens *between* state preparation and measurement: the phase the oracle plants is exploited by the diffuser to cause constructive interference on the solution. Reading this section before touching any code sets the interpretive frame for everything that follows.

### Section 2, The Toolkit: Oracle and Diffuser, Built by Hand

Rather than importing Qiskit's high-level `Grover` class, the notebook defines three functions manually:

- `phase_oracle(n, marked)`, builds $U_f$ for any list of marked bit-strings. For each marked word, it places X gates on the qubits that are 0 in that word (making the target pattern look like all-1s), applies a multi-controlled-Z (implemented as H + multi-controlled-X + H to avoid the need for a native MCZ gate), and restores the X gates.
- `diffuser(n)`, builds $D = 2|s\rangle\langle s|, I$ using the standard $H^{\otimes n}$, $X^{\otimes n}$, MCZ, $X^{\otimes n}$, $H^{\otimes n}$ circuit, with `qc.global_phase = np.pi` to correct the overall sign and make the geometry exact.
- `grover_circuit(n, marked, iters)`, applies the Hadamard layer, then stacks `iters` copies of oracle followed by diffuser.

Building these yourself, rather than calling a library method, means you know exactly what unitary is being applied and why. The `_mcz` helper is also worth studying: it shows how to implement a multi-controlled gate on $n$ qubits using only the hardware-friendly `mcx` instruction.

### Section 3, Amplitude-Level Walkthrough: Inversion About the Mean

This section is the clearest visual demonstration in the notebook. For $n = 3$ with solution `"101"`, it plots three side-by-side bar charts of amplitude values:

1. **Uniform**, all bars at $1/\sqrt{8} \approx 0.354$. The mean (dashed line) equals the bar height. Nothing is distinguishable.
2. **After oracle**, the `"101"` bar is now at $-0.354$. All magnitudes unchanged, so measurement probabilities are still flat. But the mean has dropped. The phase tag has been planted.
3. **After diffuser**, each amplitude is replaced by $2\bar{\alpha}, \alpha$. The `"101"` bar, far below the new mean, gets reflected far above it (to $\approx 0.884$). The other bars, just above the mean, are nudged down. The solution's probability has jumped from 12.5% to 78.1%.

This panel concretely illustrates why "more iterations later" refers to repeating steps 2 and 3, not step 1, and why step 1 alone gives nothing. The comment `P(101): uniform=0.125 -> after 1 iteration=0.781` is the moment students usually feel the idea click.

### Section 4, Geometric Intuition: Two Reflections Make One Rotation

This section provides the diagram that makes the algorithm "obvious." The function `draw_two_reflections` draws the unit half-circle in the $(|s'\rangle, |w\rangle)$ plane and shows three arrows: the state before the iteration, the state after the oracle (reflected down across the $|s'\rangle$ axis), and the state after the diffuser (reflected across the $|s\rangle$ line). The net effect is a rotation by $\theta = 2\arcsin\sqrt{M/N}$ toward $|w\rangle$.

`draw_rotation_frames` then shows all $k = 0, 1, \ldots, k^*$ states simultaneously, each as a coloured arrow. Numerical dots (from exact statevector simulation) are overlaid on the analytic arrows, they coincide exactly, which validates both the formula and the global-phase correction in the diffuser. If the correction were absent, the dots and arrows would drift apart.

### Section 5, How Many Iterations? The $\lfloor\frac{\pi}{4}\sqrt{N/M}\rfloor$ Rule

`plot_prob_vs_iters` draws the success probability as a continuous sinusoid (analytic formula) with the simulated values as red dots and the optimal $k^*$ as a vertical green line. The fact that the dots lie exactly on the sinusoid again confirms the clean geometry. The sinusoidal shape is crucial: it shows that success probability is periodic, not monotone, and that stopping at $k^*$ is a genuine optimisation problem, not a "more is better" situation.

### Section 6, Worked Cases

Five cases walk through the algorithm in different regimes:

**Case A ($n=2$, one solution, exact):** $N=4$, $M=1$ gives $\theta = 60°$, so $\frac{2(1)+1}{2} \times 60° = 90°$ exactly after one iteration. Success probability is exactly 1.0. This is the cleanest demonstration and a good first live example to show an audience. The circuit diagram is also plotted with `draw("mpl")` so students can see the gate sequence.

**Case B ($n=3$, one solution):** $\theta \approx 41.4°$, $k^* = 2$, $P \approx 94.5\%$. One iteration is insufficient; two gives high confidence. The rotation frame diagram shows the two-step march.

**Case C (multiple solutions, $n=4$, $M=4$):** With $M/N = 1/4$, $\theta = 60°$ and $k^* = 1$, giving $P = 1.0$ exactly. The histogram shows all four marked states (in red) dominating. This case illustrates the important principle: more solutions mean faster convergence, but a narrower window to stop in.

**Case D (over-rotation):** For $n=4$, $M=1$, $k^* = 3$, the code scans $k = 0 \ldots 12$ and plots the oscillating success probability. The green dot marks $k^* = 3$ (P ≈ 96.1%); by $k = 6$, P has collapsed to 2.0%. Red dots mark the valleys. The analogy used in the notebook, a soufflé that falls if left in the oven too long, captures the intuition perfectly.

**Case E (the $\sqrt{N}$ speedup):** A log-scale plot of oracle calls versus $n$ for both Grover ($\sim \frac{\pi}{4}\sqrt{N}$) and classical search ($\sim N/2$) for $n$ from 2 to 49. Annotation dots at $n = 3, 5, 7$ confirm the simulated $P(k^*)$ values are high. The visual gap between the two curves widening with $n$ is the most memorable illustration of what quadratic speedup means in practice.

### Section 7, Interactive Explorer

The `explore(n, target, iters)` function renders two panels, amplitude bar chart and plane diagram, for any combination of register size (2–5 qubits), target state (0–31), and iteration count (0–15). The `ipywidgets.interact` call attaches sliders so students can drag in real time. If widgets are unavailable (e.g. static Jupyter export), the function degrades gracefully to direct calls. Students are encouraged to "break" the algorithm on purpose: over-rotate it, mark many solutions, shrink to $n=2$, until the geometry feels inevitable.

### Section 8, Exercises

Four graded exercises provide increasingly challenging extensions:

- **Exercise 1 (★):** Compute $k^*$ analytically for $n=5$ and verify with the toolkit. Builds confidence in the formula.
- **Exercise 2 (★★):** Build a two-solution oracle and verify the summed probability exceeds 0.9. Reinforces that `phase_oracle` accepts a list and that multiple solutions reduce $k^*$.
- **Exercise 3 (★★★):** Find the first $k > k^*$ at which $P(k) < P(k^*)$, and connect it to the geometric picture (state has rotated past $|w\rangle$). Requires understanding the sinusoidal structure.
- **Exercise 4 (★★★★):** Implement amplitude amplification with a biased state-preparation unitary $A$ (RY rotations instead of Hadamards), adapting the diffuser to reflect about $A|0\rangle$. Show the algorithm still works and analyse how the rotation angle depends on $|\langle w | A | 0 \rangle|$. This is the full generalisation that underpins quantum amplitude estimation, quantum Monte Carlo, and many other subroutines.

Each stub returns an obviously wrong placeholder so students can tell at a glance whether their solution is live. Collapsible solution outlines are provided for self-study.

### Section 9, Summary and Next Steps

The closing section distils the algorithm to one paragraph and lists four "things to carry away," then points forward to the BBHT randomised-schedule variant (for unknown $M$), general amplitude amplification (Exercise 4 extended), and the connection back to the Bloch-sphere / Stern-Gerlach framing used earlier in the course. The invitation to return to the interactive explorer and "break it on purpose" is the right last instruction: understanding where and why a tool fails is often more instructive than seeing it succeed.

---

## Key Takeaways

- **Interference, not parallelism.** A quantum computer does not "try all answers at once." The uniform superposition carries no information; all computational work is done by the interference between oracle and diffuser. Grover is a controlled interference device.
- **The oracle tags with a phase, the diffuser converts phase to amplitude.** Together they perform one step of inversion about the mean, the marked state grows and all others shrink. Neither operation alone accomplishes anything useful; their composition is what matters.
- **Grover is a rotation in 2D.** Despite living in an exponentially large Hilbert space, the algorithm never leaves a two-dimensional plane. The oracle reflects about $|s'\rangle$, the diffuser reflects about $|s\rangle$, and two reflections compose into a rotation by $\theta = 2\arcsin\sqrt{M/N}$.
- **Stop at $k^* \approx \frac{\pi}{4}\sqrt{N/M}$, no sooner and no later.** The algorithm oscillates; it does not converge. More iterations past the optimum is as harmful as fewer. Knowing $M$, or using BBHT, is therefore necessary for practical deployment.
- **More solutions mean faster convergence, but a narrower window.** With $M = N/4$, one iteration suffices and gives exact success ($\theta = 60°$). The price is that the sinusoid completes faster, so timing errors matter more.
- **The speedup is quadratic and provably optimal.** Grover needs $O(\sqrt{N/M})$ queries; classical algorithms need $O(N/M)$. No quantum algorithm for unstructured search can do better, this is a proven lower bound, not an engineering limitation.
- **Amplitude amplification generalises everything.** Replace the Hadamard layer with any state-preparation circuit $A$, and the same geometric argument applies. This abstraction is one of the most broadly applicable primitives in quantum algorithm design.

---

## Real-World Applications

Understanding Grover's Search Algorithm is not just theoretical. Here is how it connects to active real-world problems and solutions:

- **Database and Unstructured Search**: Grover's quadratic speedup applies directly to searching unsorted databases, with applications in genomic sequence matching, fraud detection in financial transaction logs, and record retrieval.
- **Combinatorial Optimisation**: Grover-enhanced optimisation (Grover Adaptive Search) is applied to NP-hard problems including the Travelling Salesman Problem, job scheduling, and supply chain optimisation.
- **Quantum Cryptanalysis**: Grover's algorithm reduces the effective key length of symmetric encryption by half, motivating migration to AES-256 and SHA-3-512 for quantum-resistant security.
- **Drug Discovery and Molecular Docking**: Amplitude amplification variants of Grover's search are used to accelerate screening of molecular conformations in computational drug design.
- **Quantum Machine Learning**: Grover-based amplitude estimation is a subroutine in quantum Monte Carlo methods for option pricing (JPMorgan), risk analysis, and quantum-accelerated training data sampling.

---

## Further Reading & Citations

1. **Grover, L. K. (1996).** A fast quantum mechanical algorithm for database search. *Proceedings of the 28th Annual ACM Symposium on Theory of Computing (STOC)*, 212–219. [https://doi.org/10.1145/237814.237866](https://doi.org/10.1145/237814.237866), The original paper. Surprisingly readable; Grover motivates the algorithm entirely from the interference perspective.

2. **Nielsen, M. A., & Chuang, I. L. (2010).** *Quantum Computation and Quantum Information* (10th anniversary ed.). Cambridge University Press. Chapter 6 covers Grover's algorithm with full proofs of optimality, the geometric picture, and generalisation to multiple solutions. The standard reference for the field.

3. **Boyer, M., Brassard, G., Høyer, P., & Tapp, A. (1998).** Tight bounds on quantum searching. *Fortschritte der Physik*, 46(4–5), 493–505. [https://doi.org/10.1002/(SICI)1521-3978(199806)46:4/5<493::AID-PROP493>3.0.CO;2-P](https://doi.org/10.1002/(SICI)1521-3978(199806)46:4/5<493::AID-PROP493>3.0.CO;2-P), Proves the $\lfloor\frac{\pi}{4}\sqrt{N/M}\rfloor$ formula rigorously, covers the multiple-solution case, and introduces the BBHT randomised schedule for unknown $M$.

4. **Brassard, G., Høyer, P., Mosca, M., & Tapp, A. (2002).** Quantum amplitude amplification and estimation. *AMS Contemporary Mathematics*, 305, 53–74. [https://arxiv.org/abs/quant-ph/0005055](https://arxiv.org/abs/quant-ph/0005055), Introduces the full amplitude amplification framework that generalises Grover. Essential reading for Exercise 4 and for understanding how Grover fits into the broader landscape of quantum algorithms.

5. **Bennett, C. H., Bernstein, E., Brassard, G., & Vazirani, U. (1997).** Strengths and weaknesses of quantum computing. *SIAM Journal on Computing*, 26(5), 1510–1523. [https://doi.org/10.1137/S0097539796300933](https://doi.org/10.1137/S0097539796300933), Proves the $\Omega(\sqrt{N})$ lower bound: no quantum algorithm can search an unstructured database faster than Grover. Establishes that the quadratic speedup is optimal.

6. **Childs, A. M. (2022).** *Lecture Notes on Quantum Algorithms* (University of Maryland). Lecture 5: Grover's algorithm. [https://www.cs.umd.edu/~amchilds/qa/](https://www.cs.umd.edu/~amchilds/qa/), Excellent graduate-level treatment with careful proofs, alternative presentations of the geometric argument, and connections to quantum walk algorithms.

7. **Qiskit Documentation, Grover's Algorithm.** IBM Quantum Learning. [https://learning.quantum.ibm.com/course/fundamentals-of-quantum-algorithms/grovers-algorithm](https://learning.quantum.ibm.com/course/fundamentals-of-quantum-algorithms/grovers-algorithm), The official Qiskit tutorial, which provides a complementary view using the high-level `GroverOperator` class alongside conceptual explanations.

---

<!-- SEO TAGS / KEYWORDS
grover's algorithm
quantum search algorithm
qiskit grover
amplitude amplification
quantum computing tutorial
quantum interference
phase oracle
grover diffuser
inversion about the mean
quantum speedup
quadratic speedup
unstructured search
quantum amplitude
qiskit 2.5
quantum circuits python
Trinexis quantum computing
quantum computing education series by Trinexisme quantum
grover rotation geometry
quantum oracle construction
multi-controlled gate qiskit
over-rotation grover
BBHT quantum search
quantum query complexity
optimal iteration count grover
quantum superposition interference
hands-on quantum computing
quantum computing education
qiskit aer simulator
statevector simulation
quantum amplitude estimation
-->

---

## Related Notebooks in This series Series

| # | Notebook | Topic |
|---|---|---|
| 1–2 | [Demo1-2_Double_Slit_and_Stern_Gerlach.ipynb](Demo1-2_Double_Slit_and_Stern_Gerlach.ipynb) | Double-slit experiment and Stern-Gerlach, quantum foundations |
| 3 | [Demo3_QMPostulates_BraKet_Bloch.ipynb](Demo3_QMPostulates_BraKet_Bloch.ipynb) | QM postulates, Dirac notation, Bloch sphere |
| 4 | [Demo4_BlochSphere_DensityMatrix.ipynb](Demo4_BlochSphere_DensityMatrix.ipynb) | Bloch sphere and density matrices |
| 5 | [Demo5_Purity_Coherence_Entanglement.ipynb](Demo5_Purity_Coherence_Entanglement.ipynb) | Purity, coherence, and entanglement measures |
| 6 | [Demo6_Noise_and_Information_Measures.ipynb](Demo6_Noise_and_Information_Measures.ipynb) | Noise models and quantum information measures |
| 7 | [Demo7_Quantum_Gates_Demo.ipynb](Demo7_Quantum_Gates_Demo.ipynb) | Single- and multi-qubit gates |
| 8 | [Demo8_QuantumCircuits_EntanglingGates_WHT.ipynb](Demo8_QuantumCircuits_EntanglingGates_WHT.ipynb) | Quantum circuits, entangling gates, Walsh-Hadamard transform |
| 9 | [Demo9_Qiskit_Introduction.ipynb](Demo9_Qiskit_Introduction.ipynb) | Introduction to Qiskit |
| 10 | [Demo10_PennyLane_Introduction_Hands_On.ipynb](Demo10_PennyLane_Introduction_Hands_On.ipynb) | Introduction to PennyLane |
| 11 | [Demo11_Qiskit_PennyLane_Synthesis_and_Comparison.ipynb](Demo11_Qiskit_PennyLane_Synthesis_and_Comparison.ipynb) | Qiskit and PennyLane synthesis and comparison |
| 12 | [Demo12b_Qiskit_Oracles_Primitives_DJ.ipynb](Demo12b_Qiskit_Oracles_Primitives_DJ.ipynb) | Oracles, primitives, and Deutsch-Jozsa algorithm |
| 13 | [Demo13b_Bernstein_Vazirani_Qiskit.ipynb](Demo13b_Bernstein_Vazirani_Qiskit.ipynb) | Bernstein-Vazirani algorithm and phase kickback |
| 14 | [Demo14_Simons_Algorithm_Qiskit.ipynb](Demo14_Simons_Algorithm_Qiskit.ipynb) | Simon's algorithm and hidden subgroup structure |
| **15** | **Demo15_Grover_Qiskit.ipynb** *(this notebook)* | **Grover's search algorithm** |

---

*Prepared for the Trinexis Education Series in Quantum Computing, 2026. Notebook authored with Qiskit 2.5.0 / Aer 0.17.2. All simulations run on the Aer statevector and shot-based simulators.*
