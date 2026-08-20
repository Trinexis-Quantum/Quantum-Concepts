# Hands-On 10: Getting Started with PennyLane

![PennyLane](https://img.shields.io/badge/PennyLane-0.45.1-6F4BFF?style=flat-square)
![Qiskit](https://img.shields.io/badge/Qiskit-2.5.0-6929C4?style=flat-square&logo=ibm)
![Python](https://img.shields.io/badge/Python-3.8%2B-3776AB?style=flat-square&logo=python&logoColor=white)
![Jupyter](https://img.shields.io/badge/Jupyter-Notebook-F37626?style=flat-square&logo=jupyter&logoColor=white)
![License](https://img.shields.io/badge/License-MIT-22c55e?style=flat-square)
![FDP](https://img.shields.io/badge/FDP-Quantum_Computing_2026-0ea5e9?style=flat-square)

**Notebook file:** `Demo10_PennyLane_Introduction_Hands_On.ipynb`
**Course:** Quantum Computing Faculty Development Programme (FDP), June–July 2026
**Companion to:** [Demo11 — PennyLane & Qiskit Synthesis and Comparison](Demo11_Qiskit_PennyLane_Synthesis_and_Comparison.ipynb)

---

## Overview

Imagine you have spent several weeks building a car engine by hand — winding the coils yourself, machining the pistons, calibrating the fuel injectors from scratch. You now understand *exactly* how each part works. But to actually drive somewhere, you step into a fully assembled car and turn the key. That is the shift this notebook makes.

In the earlier hands-on sessions of this FDP, you assembled quantum computing from the ground up using raw NumPy: state vectors as arrays, gates as matrix multiplications, the Born rule as a norm-squared calculation, the Bloch sphere drawn by hand. That ground-up approach was deliberate — it builds the physical intuition that separates a practitioner from a user. Now you are ready to step into the car. **PennyLane** is a mature, open-source Python framework developed by [Xanadu](https://xanadu.ai/) that automates all of that bookkeeping so you can think at the level of *algorithms* rather than *linear algebra*.

What makes PennyLane special — and different from most other frameworks — is its deep commitment to **differentiable quantum computing**. Just as automatic differentiation revolutionised classical machine learning (enabling frameworks like PyTorch and TensorFlow), PennyLane brings the same idea to quantum circuits. You can compute the gradient of any circuit output with respect to any gate parameter, automatically, and in a way that is provably correct even on real quantum hardware (no classical simulation required). This notebook introduces that idea from scratch, walks you through your first differentiable circuit, shows you how entanglement looks in the framework, and closes with a full hybrid quantum–classical optimisation loop — the blueprint for virtually all Variational Quantum Algorithms and quantum machine learning.

By the end of this notebook you will have written, run, drawn, differentiated, and *trained* a quantum circuit using a professional-grade tool used in real research and industry today.

---

## Learning Objectives

After completing this notebook, you will be able to:

- **Describe** the three architectural building blocks of every PennyLane program — the **device**, the **quantum function**, and the **QNode** — and explain what role each plays.
- **Write** a quantum circuit in PennyLane using standard single- and two-qubit gates (`RX`, `RY`, `Hadamard`, `CNOT`) and execute it on a simulator.
- **Visualise** a circuit using both the text-mode `qml.draw` and the matplotlib-based `qml.draw_mpl` tools.
- **Interpret** the four main measurement return types — `qml.expval`, `qml.probs`, `qml.state`, and `qml.counts` — and explain the relationship between probabilities and squared amplitudes (the Born rule).
- **Explain** the concept of hardware-agnostic programming and change a device backend in one line without touching the circuit logic.
- **Compute gradients** of a quantum circuit's output with respect to gate parameters using `qml.grad`, and connect the result to the **parameter-shift rule**.
- **Verify** the parameter-shift rule numerically: $\frac{\partial f}{\partial \theta} = \tfrac{1}{2}[f(\theta+\tfrac{\pi}{2}) - f(\theta-\tfrac{\pi}{2})]$.
- **Construct** a Bell state in PennyLane, interpret its probability distribution as the signature of entanglement, and contrast the PennyLane syntax with Qiskit's.
- **Implement** a hybrid quantum–classical gradient-descent loop to minimise a cost function over a circuit parameter.
- **Choose** between PennyLane and Qiskit for a given task based on their design priorities and trade-offs.

---

## Background & Theory

### From Classical Bits to Quantum States

A classical bit holds either 0 or 1 — think of a light switch, firmly up or firmly down. A **qubit** can be in a *superposition* of both states simultaneously. More precisely, its state is a unit vector in a two-dimensional complex vector space:

$$|\psi\rangle = \alpha|0\rangle + \beta|1\rangle, \qquad |\alpha|^2 + |\beta|^2 = 1.$$

The coefficients $\alpha$ and $\beta$ are called **amplitudes**. They are complex numbers, and their squared magnitudes give the *probabilities* of measuring 0 or 1 respectively — the **Born rule**:

$$P(\text{outcome} = 0) = |\alpha|^2, \qquad P(\text{outcome} = 1) = |\beta|^2.$$

The **Bloch sphere** is the most useful visualisation of a single qubit: any pure state corresponds to a point on the surface of a unit sphere. The north pole is $|0\rangle$, the south pole is $|1\rangle$, and every other point is some superposition. Rotation gates move the state vector around this sphere.

### Rotation Gates and Expectation Values

The **RX gate** rotates the state vector about the $x$-axis of the Bloch sphere by an angle $\theta$:

$$RX(\theta) = \begin{pmatrix} \cos(\theta/2) & -i\sin(\theta/2) \\ -i\sin(\theta/2) & \cos(\theta/2) \end{pmatrix}.$$

Starting from $|0\rangle$ and applying $RX(\theta)$, the **expectation value** of the Pauli-$Z$ operator is:

$$\langle Z \rangle = \langle \psi | Z | \psi \rangle = \cos\theta.$$

This is a key benchmark used repeatedly throughout the notebook: a simple, analytically known formula that lets us verify everything PennyLane computes is correct.

### The Hadamard Gate and Superposition

The **Hadamard gate** is the most important single-qubit gate for creating superpositions:

$$H = \frac{1}{\sqrt{2}}\begin{pmatrix} 1 & 1 \\ 1 & -1 \end{pmatrix}, \qquad H|0\rangle = |{+}\rangle = \frac{|0\rangle + |1\rangle}{\sqrt{2}}.$$

The state $|{+}\rangle$ has equal amplitudes for 0 and 1, so both outcomes occur with equal probability 0.5. It is also the +1 eigenstate of the Pauli-$X$ operator, meaning $\langle X\rangle = 1$ on this state.

### Entanglement and the Bell State

When two qubits interact via a **CNOT gate** (controlled-NOT), they can become **entangled** — a condition where the joint state cannot be written as a product of two individual qubit states. The canonical example is the Bell state:

$$|\Phi^+\rangle = \frac{|00\rangle + |11\rangle}{\sqrt{2}}.$$

This state is produced by applying a Hadamard to qubit 0, then a CNOT with qubit 0 as control and qubit 1 as target. The resulting probability distribution is striking: only outcomes $|00\rangle$ and $|11\rangle$ can occur, each with probability 0.5, while $|01\rangle$ and $|10\rangle$ are *impossible*. Measuring one qubit instantly determines the other — a perfectly correlated pair, even though neither qubit has a definite individual state beforehand.

### Differentiable Quantum Circuits and the Parameter-Shift Rule

The heart of PennyLane's design is **automatic differentiation of quantum circuits**. For a circuit parameterised by a rotation angle $\theta$, the exact partial derivative can be computed using two circuit evaluations at *shifted* parameters:

$$\frac{\partial \langle O \rangle}{\partial \theta} = \frac{1}{2}\Big[\langle O\rangle\!\left(\theta + \frac{\pi}{2}\right) - \langle O\rangle\!\left(\theta - \frac{\pi}{2}\right)\Big].$$

This is the **parameter-shift rule**, first established by Mitarai et al. (2018) and Schuld et al. (2019). Its power lies in what it does *not* require: it needs no access to the circuit's internal state, no classical back-propagation through the simulator, no analytic formulas written by hand. It works by running the circuit twice with different inputs — exactly the kind of query a real quantum computer can answer. This means gradients of quantum circuits can be computed *on real hardware*, not just simulators, making them first-class citizens in any optimisation or machine-learning pipeline.

### Hybrid Quantum–Classical Optimisation

A **Variational Quantum Algorithm (VQA)** combines a parameterised quantum circuit (the "quantum layer") with a classical optimiser (e.g., gradient descent). The loop is:

1. Run the circuit with current parameters $\boldsymbol{\theta}$ to compute a cost $C(\boldsymbol{\theta})$.
2. Use the parameter-shift rule to compute the gradient $\nabla_{\boldsymbol{\theta}} C$.
3. Update: $\boldsymbol{\theta} \leftarrow \boldsymbol{\theta} - \eta \nabla_{\boldsymbol{\theta}} C$.
4. Repeat until convergence.

This is structurally identical to training a neural network — the quantum circuit plays the role of the neural network's forward pass, and the parameter-shift rule plays the role of back-propagation. The notebook demonstrates this with a toy single-qubit example; the same architecture scales to chemistry (VQE), combinatorial optimisation (QAOA), and quantum machine learning.

---

## Prerequisites

**Quantum computing knowledge:**
- Single-qubit states: Dirac notation $|0\rangle$, $|1\rangle$, superposition, inner products (covered in Demo3)
- The Bloch sphere and rotation gates (covered in Demo3 and Demo4)
- Expectation values and the Born rule (covered in Demo4 and Demo5)
- Basic entanglement and the CNOT gate (covered in Demo5 and Demo8)

**Python and software tools:**
- Comfortable with Python 3: functions, loops, NumPy arrays
- Basic familiarity with Matplotlib for plotting
- Some exposure to Qiskit is helpful but not required (covered in Demo9)

**Mathematics:**
- Complex numbers and vectors
- Basic calculus: derivatives, the chain rule (to appreciate automatic differentiation)
- No knowledge of gradients or machine learning is assumed — those concepts are developed from scratch here

**Environment:**
- Google Colab (recommended — no local setup needed) or a local Python 3.8+ environment
- PennyLane and Qiskit are installed automatically by the first cell

---

## Notebook Walkthrough

### Section 0 — Setup

The opening cell installs PennyLane and Qiskit using a guard that checks whether each package is already present before calling `pip`. This makes the cell safe to re-run without redundant network traffic. The second setup cell introduces a subtlety that trips up many beginners: PennyLane ships its *own* NumPy wrapper (`pennylane.numpy`, imported as `pnp`) alongside ordinary NumPy (`np`). The PennyLane wrapper is "gradient-aware" — it tracks operations on arrays for automatic differentiation. Any parameter you intend to optimise or differentiate must be a `pnp` array; ordinary `np` is used for everything else (plotting, index arrays, ranges).

### Section 1 — The Big Picture: Three Building Blocks

Before any code, this section establishes the conceptual architecture. A PennyLane program always has three parts:

- **Device**: where the circuit runs — a software simulator or a cloud-connected real quantum processor. Changing the device does not change the circuit.
- **Quantum function**: an ordinary Python function that applies gates in sequence and returns a measurement. The function body reads like a human description of the circuit.
- **QNode**: the result of binding a quantum function to a device via the `@qml.qnode(dev)` decorator. From the outside, a QNode behaves exactly like a regular Python function — numbers go in, numbers come out — which is what allows it to be embedded in any classical optimisation loop.

The ASCII diagram in the notebook is worth studying: it shows the clean separation between the circuit interior (gate operations) and the classical exterior (numbers flowing in and out).

### Section 2 — Your First Circuit

The first live circuit is deliberately the simplest possible: one qubit, one gate (`RX(theta)`), one measurement (`expval(PauliZ)`). The notebook evaluates it at $\theta = 0$, $\pi/2$, and $\pi$ and confirms the output matches $\cos\theta$ exactly. This serves two purposes: it verifies the framework is working correctly, and it anchors the unfamiliar PennyLane syntax to a result the student already knows from the Bloch sphere session.

The `qml.draw` and `qml.draw_mpl` tools are introduced immediately after. Getting in the habit of drawing circuits before running them is professional practice — it catches gate ordering mistakes and wiring errors that are otherwise invisible in the code.

### Section 3 — How PennyLane Reports Results

This section reveals that PennyLane's measurement system is more flexible than it first appears. By changing only the `return` statement, you get four qualitatively different views of the same final state:

- `qml.expval(obs)` — a single real number, the quantum average of an observable.
- `qml.probs(wires=...)` — a probability vector over all basis states (the Born rule directly).
- `qml.state()` — the full complex state vector (simulator only; a real device cannot return this).
- `qml.counts()` / `qml.sample()` — individual shot outcomes, mimicking a real device.

The notebook prepares $|{+}\rangle$ with a Hadamard and queries all four, then explicitly shows that the probabilities in (b) are the squared magnitudes of the amplitudes in (c). Seeing $|0.707|^2 = 0.5$ in actual output is the Born rule made viscerally concrete.

The `@qml.set_shots` transform is introduced to simulate finite sampling noise. With 2000 shots on an equal superposition, the empirical frequencies come out close to — but not exactly — 0.5/0.5. This is the statistical reality of real quantum hardware, and understanding it is essential before working with any physical device.

### Section 4 — Write Once, Run Anywhere

This short but important section demonstrates hardware-agnostic programming. The *exact same* Bell-state circuit function is passed to `default.qubit` (PennyLane's pure-Python simulator) and to `lightning.qubit` (a faster C++ simulator), producing identical results. The only change in the code is the string `"default.qubit"` versus `"lightning.qubit"`. In a real project, you would swap in an IBM, Amazon Braket, or Xanadu hardware backend the same way. The circuit logic itself never changes. This portability is a first-class design goal of PennyLane, not an afterthought.

### Section 5 — Differentiable Quantum Circuits

This is the conceptual centrepiece of the notebook. The `qml.grad` function accepts a QNode and returns a new function that computes its gradient. The result at $\theta = \pi/4$ is $-\sin(\pi/4) \approx -0.707$ — matching the analytic derivative of $\cos\theta$ to five decimal places, with no derivative formula written by the user.

The notebook then plots the circuit output and its PennyLane-computed gradient against the analytic curves $\cos\theta$ and $-\sin\theta$ across the full range $[0, 2\pi]$. The PennyLane curves sit exactly on top of the analytic curves — a visually compelling proof that the framework is not approximating the gradient but computing it exactly.

The *why* is then explained with the parameter-shift rule. A code cell implements the rule by hand — two circuit evaluations at $\theta \pm \pi/2$, combined with a factor of $1/2$ — and confirms it agrees with `qml.grad` and the analytic formula to six decimal places. This makes the abstract formula tangible: the "automatic" gradient is not magic; it is two circuit evaluations with a shift.

### Section 6 — Entanglement, the Framework Way

The Bell state from the earlier entanglement module appears again, now in three lines of PennyLane: `Hadamard` followed by `CNOT`. The notebook returns both `qml.state()` and `qml.probs()`, confirming the state vector $\tfrac{1}{\sqrt{2}}(|00\rangle + |11\rangle)$ and the probability distribution $P(00) = P(11) = 0.5$, $P(01) = P(10) = 0$. A side-by-side figure shows the circuit diagram and the bar chart of probabilities, making the signature of entanglement visually obvious: only the fully correlated outcomes survive.

### Section 7 — Hybrid Quantum–Classical Training Loop

Every element introduced so far — the QNode, `pnp` arrays, `qml.grad`, the cost function pattern — comes together in a 60-step gradient descent loop. The cost function is $\langle Z\rangle(\theta) = \cos\theta$, which is minimised (pushed toward $-1$) by rotating the qubit from $|0\rangle$ toward $|1\rangle$. Starting from $\theta_0 = 0.1$, the `GradientDescentOptimizer` discovers $\theta^* \approx \pi$ entirely from gradient information. The convergence plot shows a smooth descent from near $+1$ to exactly $-1$ over 60 steps.

This loop is structurally identical to training a one-layer neural network. Replacing the single-parameter RX circuit with a deeper, multi-parameter circuit and replacing $\langle Z\rangle$ with a chemically meaningful cost function produces the Variational Quantum Eigensolver; replacing it with a combinatorial cost produces QAOA; connecting it to a classical data pipeline produces quantum machine learning. The student has just seen the universal skeleton.

### Section 8 — PennyLane vs. Qiskit

Both frameworks build the same Bell state, and the results agree ($P(00) = P(11) = 0.5$). The syntax comparison reveals structural similarities — both frameworks apply gates sequentially to named qubits and return measurements — but also a practical difference: Qiskit uses little-endian bit ordering for its basis-state labels, which becomes significant for non-symmetric states. The comparison table summarises the trade-offs: PennyLane's strengths are gradients, hardware-agnosticism, and ML-native integration; Qiskit's strengths are IBM hardware access, a larger general user base, and a gentler on-ramp for absolute beginners. The closing rule of thumb is deliberately non-partisan: they are complementary, many researchers use both, and PennyLane can even run circuits on IBM hardware via a plugin.

### Section 9 — Graded Exercises

Three exercises test the student at increasing difficulty:

- **Exercise 1 (★):** Prepare $|{+}\rangle$ with a Hadamard and confirm $\langle X\rangle = +1$. One gate to add; the measurement is pre-set.
- **Exercise 2 (★★):** Build an `RY(theta)` circuit, sweep over $\theta \in [0, 2\pi]$, plot $\langle Z\rangle = \cos\theta$, and identify the first zero crossing ($\theta = \pi/2$).
- **Exercise 3 (★★★):** Define a squared-error cost $({\langle Z\rangle - 0.5})^2$ and run gradient descent to find the angle $\theta = \arccos(0.5) = \pi/3$ at which $\langle Z\rangle = 0.5$. This generalises the training loop from a hard target ($-1$) to an arbitrary intermediate value.

Each exercise has a hidden solution in a `<details>` block; students are encouraged to attempt the exercise first, then reveal the solution for comparison.

---

## Key Takeaways

- **The QNode abstraction is what makes quantum-classical integration seamless.** A QNode looks like an ordinary Python function to the classical world, which means it can be plugged into any loop, optimiser, or ML library without special handling.

- **The Born rule is not an external postulate in PennyLane — it is baked into the measurement API.** The `qml.probs` return type automatically squares and normalises the amplitudes; seeing $|0.707|^2 = 0.5$ in the output is the Born rule made automatic.

- **The parameter-shift rule gives *exact* gradients using only circuit evaluations.** This is not an approximation like finite differences. It works on real quantum hardware where classical back-propagation is impossible, making quantum circuits genuinely trainable.

- **Hardware-agnostic programming is a first-class feature, not a nice-to-have.** Swapping one string changes the execution backend from a Python simulator to a C++ simulator to a real quantum processor — the circuit code never changes.

- **Entanglement has a clear observable signature.** The Bell state's probability distribution — only $|00\rangle$ and $|11\rangle$, never $|01\rangle$ or $|10\rangle$ — is the direct fingerprint of quantum correlation, visible immediately in the `qml.probs` output.

- **The hybrid quantum–classical training loop is the universal template for quantum algorithms.** VQE, QAOA, quantum neural networks, and quantum kernel methods all share the same architecture: parameterised circuit, cost function, gradient, optimiser update. This notebook shows it at the simplest possible scale.

- **PennyLane and Qiskit are complementary tools.** Knowing when to use each — gradients and ML for PennyLane, IBM hardware and gate-level circuits for Qiskit — is a practical skill for any quantum computing researcher or engineer.

---

## Further Reading & Citations

1. **Bergholm, V., Izaac, J., Schuld, M., et al. (2018).** PennyLane: Automatic differentiation of hybrid quantum-classical computations. *arXiv preprint arXiv:1811.04968*. [https://arxiv.org/abs/1811.04968](https://arxiv.org/abs/1811.04968)
   *The original PennyLane paper. Describes the framework architecture, the QNode abstraction, and the integration of quantum circuits with classical automatic differentiation.*

2. **Mitarai, K., Negoro, M., Kitagawa, M., & Fujii, K. (2018).** Quantum circuit learning. *Physical Review A, 98*(3), 032309. [https://arxiv.org/abs/1803.00745](https://arxiv.org/abs/1803.00745)
   *One of the foundational papers establishing that parameterised quantum circuits can be trained using gradient methods. Introduces the parameter-shift rule in the context of quantum circuit learning.*

3. **Schuld, M., Bergholm, V., Gogolin, C., Izaac, J., & Killoran, N. (2019).** Evaluating analytic gradients on quantum hardware. *Physical Review A, 99*(3), 032331. [https://arxiv.org/abs/1811.11184](https://arxiv.org/abs/1811.11184)
   *Rigorously establishes the parameter-shift rule for hardware-compatible gradient evaluation. Proves that the shift rule gives exact (not approximate) derivatives for rotation gates.*

4. **Cerezo, M., Arrasmith, A., Babbush, R., et al. (2021).** Variational quantum algorithms. *Nature Reviews Physics, 3*(9), 625–644. [https://arxiv.org/abs/2012.09265](https://arxiv.org/abs/2012.09265)
   *A comprehensive review of the VQA landscape — VQE, QAOA, quantum neural networks — built on exactly the hybrid optimisation loop introduced in Section 7 of this notebook.*

5. **Biamonte, J., Wittek, P., Pancotti, N., Rebentrost, P., Wiebe, N., & Lloyd, S. (2017).** Quantum machine learning. *Nature, 549*(7671), 195–202. [https://arxiv.org/abs/1611.09347](https://arxiv.org/abs/1611.09347)
   *The seminal survey that brought quantum machine learning into mainstream discussion. Provides the conceptual motivation for differentiable quantum circuits and quantum–classical hybrid models.*

6. **Nielsen, M. A., & Chuang, I. L. (2010).** *Quantum Computation and Quantum Information* (10th anniversary edition). Cambridge University Press.
   *The standard graduate-level textbook. Chapters 1–4 cover qubits, quantum gates, the circuit model, and entanglement at a depth that fully supports all concepts in this notebook.*

7. **Schuld, M., & Petruccione, F. (2021).** *Machine Learning with Quantum Computers* (2nd edition). Springer.
   *A modern textbook that bridges quantum computing and machine learning, covering variational circuits, quantum kernels, and the differentiable programming paradigm that PennyLane implements.*

8. **Peruzzo, A., McClean, J., Shadbolt, P., et al. (2014).** A variational eigenvalue solver on a photonic chip. *Nature Communications, 5*(1), 4213. [https://arxiv.org/abs/1304.3061](https://arxiv.org/abs/1304.3061)
   *The original VQE paper — the first experimental demonstration of a hybrid quantum–classical optimisation loop, exactly the pattern introduced in Section 7 of this notebook.*

---

<!-- 
SEO TAGS / KEYWORDS — for repository indexing and discoverability

pennylane tutorial, pennylane introduction, pennylane qnode, pennylane quantum circuit,
pennylane getting started, differentiable quantum computing, parameter shift rule,
quantum automatic differentiation, hybrid quantum classical, variational quantum algorithm,
VQA tutorial, quantum machine learning, QML tutorial, quantum gradient descent,
pennylane vs qiskit, quantum circuit training, bell state pennylane, quantum entanglement demo,
pennylane expval probs state, quantum computing FDP, pennylane GradientDescentOptimizer,
qml.grad tutorial, quantum circuit drawing, pennylane draw, pennylane device backend,
default.qubit lightning.qubit, hardware agnostic quantum, pennylane numpy, trainable quantum circuit,
quantum optimizer python, born rule probabilities, superposition measurement, hadamard gate demo,
RX gate bloch sphere, pennylane colab, quantum computing workshop, xanadu pennylane,
quantum software framework, quantum parameter optimization, shots simulation, quantum sampling
-->

---

## Related Notebooks in This FDP Series

| # | Notebook | Topic |
|---|----------|-------|
| 1–2 | [Demo1-2 — Double Slit and Stern–Gerlach](Demo1-2_Double_Slit_and_Stern_Gerlach_FDP.ipynb) | Wave–particle duality, foundational quantum experiments |
| 3 | [Demo3 — QM Postulates, Bra-Ket, and the Bloch Sphere](Demo3_QMPostulates_BraKet_Bloch.ipynb) | Dirac notation, state vectors, single-qubit geometry |
| 4 | [Demo4 — Bloch Sphere and Density Matrices](Demo4_BlochSphere_DensityMatrix.ipynb) | Mixed states, density operator formalism |
| 5 | [Demo5 — Purity, Coherence, and Entanglement](Demo5_Purity_Coherence_Entanglement.ipynb) | Quantum information measures, entanglement criteria |
| 6 | [Demo6 — Noise and Information Measures](Demo6_Noise_and_Information_Measures.ipynb) | Quantum channels, decoherence, entropy |
| 7 | [Demo7 — Quantum Gates](Demo7_Quantum_Gates_Demo.ipynb) | Single- and multi-qubit gate zoo |
| 8 | [Demo8 — Quantum Circuits, Entangling Gates, and WHT](Demo8_QuantumCircuits_EntanglingGates_WHT.ipynb) | Circuit composition, Walsh–Hadamard transform |
| 9 | [Demo9 — Qiskit Introduction](Demo9_Qiskit_Introduction.ipynb) | IBM Qiskit framework, circuit construction, noise models |
| **10** | **Demo10 — PennyLane Introduction** *(this notebook)* | **PennyLane, differentiable circuits, hybrid optimisation** |
| 11 | [Demo11 — Qiskit & PennyLane Synthesis and Comparison](Demo11_Qiskit_PennyLane_Synthesis_and_Comparison.ipynb) | Deep framework comparison, interoperability |
| 12 | [Demo12b — Qiskit Oracles, Primitives, and Deutsch–Jozsa](Demo12b_Qiskit_Oracles_Primitives_DJ.ipynb) | Oracle-based algorithms, quantum speedup |
| 13 | [Demo13b — Bernstein–Vazirani Algorithm](Demo13b_Bernstein_Vazirani_Qiskit.ipynb) | Hidden bit-string recovery, interference |
| 14 | [Demo14 — Simon's Algorithm](Demo14_Simons_Algorithm_Qiskit.ipynb) | Exponential speedup, period finding |
| 15 | [Demo15 — Grover's Search Algorithm](Demo15_Grover_Qiskit_FDP.ipynb) | Quadratic speedup, amplitude amplification |

---

*Prepared for the Quantum Computing Faculty Development Programme, June–July 2026.*
*Notebook runs on Google Colab — use Runtime → Run all for a complete end-to-end execution.*
*PennyLane documentation: [https://docs.pennylane.ai/](https://docs.pennylane.ai/) | PennyLane demos: [https://pennylane.ai/qml/](https://pennylane.ai/qml/)*
