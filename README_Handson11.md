# ⚛️ Hands-on 11 — Qiskit vs. PennyLane: Synthesis & Side-by-Side Comparison

[![Qiskit](https://img.shields.io/badge/Qiskit-2.5.0-6929C4?logo=ibm&logoColor=white)](https://qiskit.org/)
[![PennyLane](https://img.shields.io/badge/PennyLane-0.45.1-3C7EBB?logo=python&logoColor=white)](https://pennylane.ai/)
[![Python](https://img.shields.io/badge/Python-3.9%2B-3776AB?logo=python&logoColor=white)](https://www.python.org/)
[![Jupyter](https://img.shields.io/badge/Jupyter-Notebook-F37626?logo=jupyter&logoColor=white)](https://jupyter.org/)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
[![Colab](https://img.shields.io/badge/Open%20in-Colab-F9AB00?logo=googlecolab&logoColor=white)](https://colab.research.google.com/)

> **Capstone of the Software-Ecosystem Session** — Quantum Computing Education Series by Trinexis

---

## Overview

If you have ever learned two human languages and then held the same conversation in both, you know something remarkable: the ideas are identical, but the idioms are entirely different. That is precisely the experience this notebook is designed to give you about quantum software. By now you have spent time with **Qiskit** — IBM's circuit-first framework — and **PennyLane** — Xanadu's differentiable-programming framework. This notebook places them side by side on *identical* tasks and lets the running numbers speak for themselves.

The central insight is deceptively simple: both frameworks simulate the same quantum mechanics using the same mathematics. A Bell state prepared in Qiskit and a Bell state prepared in PennyLane are described by the same wavefunction, produce the same measurement probabilities, and obey the same physical laws. What differs is the *bookkeeping* — how you express the circuit, how you invoke the computation, and how you obtain gradients when you need to train a quantum circuit. Think of Qiskit as a precision machine tool: powerful, hardware-aware, built for manufacturing. Think of PennyLane as a programmable calculator with built-in calculus: elegant for exploration, native in the language of optimisation. Neither is universally superior; they were built to solve different first problems.

This notebook makes that abstract argument concrete. You will watch two training curves — one computed by hand with the parameter-shift rule inside Qiskit, the other computed automatically by PennyLane's autodiff engine — converge to *the same answer, at machine-precision, step for step*. That numerical coincidence is not magic: it is a theorem. Understanding why will give you the deepest possible intuition for how modern quantum software actually works.

---

## Learning Objectives

After completing this notebook, you will be able to:

- Express the **same quantum circuit** (Bell, GHZ) in both Qiskit and PennyLane syntax and explain the structural differences between the two approaches.
- Describe the **two core abstractions** — the `QuantumCircuit` object (Qiskit) and the `QNode` function (PennyLane) — and articulate why the distinction matters for downstream workflows.
- Implement the **parameter-shift rule manually** in Qiskit to compute analytical gradients of an expectation value, and explain each step of the derivation.
- Use **PennyLane's `qml.grad`** to obtain the same gradient automatically, and explain why the two results are numerically identical.
- Run a **variational optimisation loop** (gradient descent on ⟨Z⟩) in both frameworks and verify convergence to the same solution.
- Apply a structured **decision framework** to select the right tool for a given quantum computing task.
- Extend both implementations to generalised **n-qubit GHZ states** and verify cross-framework agreement.

---

## Background & Theory

### The Quantum Gates at Play

Before comparing frameworks, it helps to understand the physics. This notebook uses two foundational quantum gates.

**The Hadamard gate (H)** is the quantum equivalent of a fair coin flip. If you hand it a qubit in the definite state |0⟩ (think: "heads"), it produces an *equal superposition* — a state that is simultaneously half |0⟩ and half |1⟩:

$$H|0\rangle = \frac{|0\rangle + |1\rangle}{\sqrt{2}}$$

The factor $1/\sqrt{2}$ is the *probability amplitude*, not a probability. The actual probability of measuring |0⟩ or |1⟩ is the square: $(1/\sqrt{2})^2 = 1/2$ each. Superposition is not ignorance — the qubit is genuinely in both states until measured.

**The CNOT gate** is the quantum equivalent of "if the first switch is on, flip the second." If qubit 0 is |1⟩, it flips qubit 1; if qubit 0 is |0⟩, it does nothing. When the first qubit is in superposition, the CNOT produces *entanglement* — a joint state that cannot be described by assigning separate states to each qubit independently.

### The Bell State

Applying H to qubit 0 and then CNOT(0→1) creates the **Bell state** $|\Phi^+\rangle$:

$$|\Phi^+\rangle = \frac{|00\rangle + |11\rangle}{\sqrt{2}}$$

This state has no classical analogue. Measuring either qubit gives 0 or 1 with equal probability, but the two outcomes are *perfectly correlated*: if you see 00, your partner always sees 00; if you see 11, your partner always sees 11. This correlation is non-classical (it violates Bell inequalities) and is the physical resource behind quantum teleportation and quantum key distribution.

### The GHZ State

Extending the Bell construction to $n$ qubits gives the **Greenberger–Horne–Zeilinger (GHZ) state**:

$$|GHZ_n\rangle = \frac{|0\rangle^{\otimes n} + |1\rangle^{\otimes n}}{\sqrt{2}}$$

The circuit recipe is H on qubit 0, then a chain of CNOT gates: CNOT(0,1), CNOT(1,2), ..., CNOT(n-2, n-1). GHZ states are the maximally entangled generalisations of Bell states and appear throughout quantum error correction, quantum networks, and tests of multi-partite entanglement.

### The RX Rotation and ⟨Z⟩

A **rotation-X gate** $RX(\theta)$ rotates a qubit's state around the X-axis of the Bloch sphere by angle $\theta$:

$$RX(\theta)|0\rangle = \cos\!\left(\frac{\theta}{2}\right)|0\rangle - i\sin\!\left(\frac{\theta}{2}\right)|1\rangle$$

The **expectation value of the Pauli-Z operator** on this state is:

$$\langle Z \rangle = \cos(\theta)$$

At $\theta = 0$, the qubit sits at the north pole of the Bloch sphere: $\langle Z \rangle = +1$. At $\theta = \pi$, it sits at the south pole: $\langle Z \rangle = -1$. Gradient descent on the cost $\mathcal{L}(\theta) = \langle Z \rangle$ will therefore drive $\theta \to \pi$.

### The Parameter-Shift Rule

Quantum circuits are not differentiable in the classical automatic-differentiation sense: you cannot "backpropagate through a measurement." However, for gates of the form $G(\theta) = e^{-i\theta P/2}$ (where $P$ is a Pauli operator), the gradient of any expectation value with respect to $\theta$ obeys the exact **parameter-shift rule**:

$$\frac{\partial \langle O \rangle}{\partial \theta} = \frac{1}{2}\Bigl[\langle O \rangle_{\theta + \pi/2} - \langle O \rangle_{\theta - \pi/2}\Bigr]$$

This is an *exact* formula, not a finite-difference approximation. It requires two circuit evaluations — one with $\theta$ shifted by $+\pi/2$, one with $\theta$ shifted by $-\pi/2$ — and returns the exact gradient. PennyLane's `qml.grad` applies this rule automatically behind the scenes. Qiskit requires you to write the two shifted evaluations by hand. Both produce the identical numerical gradient, which is why the two training curves in this notebook lie on top of each other.

---

## Prerequisites

| Category | What You Need |
|---|---|
| **Quantum concepts** | Qubits, superposition, entanglement, measurement, Bloch sphere (Demos 3–5 of this series) |
| **Quantum gates** | H, CNOT, RX, Pauli operators (Demo 7–8) |
| **Qiskit basics** | `QuantumCircuit`, `Parameter`, `Statevector`, `StatevectorEstimator`, `SparsePauliOp` (Demo 9) |
| **PennyLane basics** | `qml.device`, `@qml.qnode`, `qml.grad`, `GradientDescentOptimizer` (Demo 10) |
| **Python** | NumPy array operations, defining functions, for-loops |
| **Optimisation** | Gradient descent concept (step in the direction of steepest descent) |
| **Software** | Python 3.9+, Jupyter or Google Colab; the notebook auto-installs `qiskit`, `pennylane`, `pylatexenc` |

---

## Notebook Walkthrough

### Section 0 — Setup

The setup cell checks for each required package using `importlib.util.find_spec` before attempting installation — a best practice that avoids redundant pip calls in repeated Colab runs. After installation, version strings are printed so you can verify you are running compatible releases (Qiskit 2.5.0, PennyLane 0.45.1). This section is administrative but worth reading: noticing *how* the dependency check is written is itself a small software-engineering lesson.

### Section 1 — Same Circuit, Two Languages: Bell and GHZ States

This section is the clearest possible statement of the notebook's thesis. The *exact same physical circuit* — H on qubit 0, CNOT(0,1) — is written first in Qiskit and then in PennyLane. Qiskit produces a `QuantumCircuit` *object* that you pass to `Statevector.from_instruction()`. PennyLane wraps the same gate sequence as a Python *function* (the QNode) that you call directly. The output probabilities — `{'00': 0.5, '11': 0.5}` — are identical.

The GHZ cell extends this to three qubits with a chain of two CNOTs. Again, the probabilities agree exactly. The commentary after these cells calls attention to the architectural difference: Qiskit thinks in terms of *circuit objects + primitives*; PennyLane thinks in terms of *quantum functions*. That difference in mental model propagates into every subsequent choice each framework makes.

**Why this matters:** Before comparing frameworks, you must be sure they are solving the same problem. Section 1 provides that verification in the most direct way possible — by running both and checking the numbers agree.

### Section 2 — Same Task, Two Philosophies: Training a Qubit Rotation

This is the intellectual core of the notebook. The optimisation problem is deliberately simple — minimise $\langle Z \rangle$ for $RX(\theta)|0\rangle$, starting at $\theta_0 = 0.10$, step size $0.3$, 60 iterations — so that *nothing* obscures the comparison between the two approaches.

**Qiskit side:** A symbolic `Parameter('theta')` is embedded in the circuit. For each gradient step, the circuit is bound at $\theta + \pi/2$ and $\theta - \pi/2$, the `StatevectorEstimator` evaluates the expectation value at each, and the parameter-shift formula gives the gradient. The update rule $\theta \leftarrow \theta - \alpha \cdot \nabla_\theta \langle Z \rangle$ is applied by hand. This is explicit, transparent, and instructive: you see every moving part.

**PennyLane side:** The QNode returns `qml.expval(qml.PauliZ(0))`. A `GradientDescentOptimizer` with `stepsize=0.3` wraps `qml.grad` and handles the parameter-shift rule invisibly. The loop calls `opt.step(cost_fn, theta)` and the framework does the rest.

**The result:** Both trajectories converge to $\theta = \pi$ (i.e., $\langle Z \rangle = -1$) and the maximum absolute difference between the two full 61-point trajectories is $6 \times 10^{-16}$ — below double-precision floating-point rounding error. The plot in Section 2 shows both curves lying exactly on top of each other.

**The lesson:** PennyLane's autodiff engine *is* the parameter-shift rule, applied automatically. Choosing PennyLane over Qiskit for variational algorithms does not change the mathematics — it offloads the bookkeeping.

### Section 3 — A Deeper Comparison Table

The comparison table in Section 3 synthesises everything from Demos 9 and 10 into a single structured view across eight dimensions: primary focus, core abstraction, hardware stance, gradient support, ML framework integration, typical entry point, visualisation, and ecosystem extensions. Reading across each row builds a coherent picture of the two frameworks' different design philosophies.

Crucially, the table does not declare a winner. It is a tool for *informed selection*, not advocacy. The final row — governance — is a reminder that open-source communities (Xanadu) and corporate-backed projects (IBM) make different trade-offs about stability, vendor lock-in, and long-term support.

### Section 4 — Which Tool, When? A Practical Decision Framework

Section 4 translates the comparison table into actionable guidance using a series of "if your situation is X, use Y" rules. The framework covers: hardware targeting (IBM → Qiskit; hardware-agnostic → PennyLane), ML integration (PyTorch/TF/JAX → PennyLane), production error mitigation (Qiskit Runtime), and algorithm libraries (both have one — choose based on your existing stack).

The section closes with the key pragmatic observation that many production research teams use *both frameworks simultaneously* — Qiskit for hardware runs and error mitigation, PennyLane for model development and gradient-based training. The frameworks are complementary, not competing.

### Section 5 — Exercises

**Exercise 1 (single star)** modifies the training loop to minimise $(\langle Z \rangle - 0)^2$ instead of $\langle Z \rangle$, targeting $\langle Z \rangle = 0$ (the equatorial superposition state). This requires applying the chain rule on the Qiskit side: the gradient of $f(\theta)^2$ is $2 f(\theta) f'(\theta)$. On the PennyLane side, `qml.grad` handles the chain rule automatically. The exercise reinforces that framework choice affects *how much calculus you implement by hand*, not the calculus itself. The hint and hidden solution guide students who get stuck.

**Exercise 2 (two stars)** asks students to implement a parameterised `n`-qubit GHZ state function in both frameworks and verify that the two agree for $n = 2, 3, 4, 5$. The hint (H on qubit 0, then CX(i, i+1) for i = 0..n-2) is sufficient for students who completed Section 1. This exercise builds the generalisation habit and confirms cross-framework consistency at scale.

### Section 6 — Wrap-up & Further Resources

The closing section states the course's core philosophical claim: *frameworks change the bookkeeping, not the physics*. Every result in this notebook was, in principle, computable with NumPy alone. What frameworks provide is hardware access, a tested library, visualisation, and automation of repetitive mathematical operations. Choosing a framework is an engineering decision, not a physics one.

Official documentation links for both frameworks are provided, and a forward pointer explains that the parameter-shift / `Estimator` / `expval` machinery from both notebooks feeds directly into the next hands-on notebook on the Bell inequality (CHSH).

---

## Key Takeaways

- **Identical physics, different syntax.** Every circuit in this notebook encodes the same quantum mechanics in both frameworks. Qiskit and PennyLane agree at machine precision on every probability and expectation value.
- **The QNode/circuit-object distinction is architectural, not cosmetic.** Qiskit's circuit-object model makes explicit transpilation, hardware targeting, and primitive dispatch natural. PennyLane's function model makes composition, differentiation, and ML integration natural. Neither is a bug; both reflect deliberate design choices.
- **The parameter-shift rule is an exact gradient formula, not a numerical approximation.** Unlike finite differences, it gives the true gradient of an expectation value with only two circuit evaluations, regardless of the number of parameters.
- **PennyLane's autodiff automates the parameter-shift rule; it does not replace it.** The two training curves coincide at $6 \times 10^{-16}$ — below floating-point rounding error — because they compute the same formula.
- **GHZ states generalise Bell states and scale linearly in circuit depth.** The circuit for an $n$-qubit GHZ state requires exactly $n-1$ CNOT gates after one Hadamard — a depth that grows modestly while producing exponentially complex entanglement.
- **Framework choice is an engineering decision.** For IBM hardware + error mitigation, prefer Qiskit. For ML integration + hardware agnosticism + rapid prototyping of variational algorithms, prefer PennyLane. In practice, many teams use both.
- **Frameworks do not change quantum mechanics.** They change how much bookkeeping you do by hand. The most important skill is understanding the underlying mathematics, so you can read either framework fluently.

---

## Further Reading & Citations

1. **Nielsen, M. A. & Chuang, I. L.** (2010). *Quantum Computation and Quantum Information* (10th anniversary ed.). Cambridge University Press. — The standard graduate-level textbook; Chapters 1–2 cover qubits, gates, and the circuit model used throughout this notebook.

2. **Bergholm, V., Izaac, J., Schuld, M., Gogolin, C., Ahmed, S., Ajith, V., ... & Killoran, N.** (2022). PennyLane: Automatic differentiation of hybrid quantum-classical computations. *arXiv:1811.04968*. [https://arxiv.org/abs/1811.04968](https://arxiv.org/abs/1811.04968) — The primary reference for PennyLane's design, QNode abstraction, and automatic differentiation backend.

3. **Mitarai, K., Negoro, M., Kitagawa, M., & Fujii, K.** (2018). Quantum circuit learning. *Physical Review A*, 98(3), 032309. [https://doi.org/10.1103/PhysRevA.98.032309](https://doi.org/10.1103/PhysRevA.98.032309) — Introduces the parameter-shift rule as an exact gradient formula for quantum circuits; the theoretical foundation of Section 2.

4. **Schuld, M., Bergholm, V., Gogolin, C., Izaac, J., & Killoran, N.** (2019). Evaluating analytic gradients on quantum hardware. *Physical Review A*, 99(3), 032331. [https://doi.org/10.1103/PhysRevA.99.032331](https://doi.org/10.1103/PhysRevA.99.032331) — Proves that the parameter-shift rule yields exact gradients on real quantum hardware, not just simulators.

5. **Cerezo, M., Arrasmith, A., Babbush, R., Benjamin, S. C., Endo, S., Fujii, K., ... & Coles, P. J.** (2021). Variational quantum algorithms. *Nature Reviews Physics*, 3(9), 625–644. [https://doi.org/10.1038/s42254-021-00348-9](https://doi.org/10.1038/s42254-021-00348-9) — Comprehensive review of variational quantum algorithms (VQAs), the family of methods that Section 2's optimisation loop belongs to.

6. **Greenberger, D. M., Horne, M. A., & Zeilinger, A.** (1989). Going beyond Bell's theorem. In *Bell's Theorem, Quantum Theory and Conceptions of the Universe* (pp. 69–72). Springer. — The original paper introducing GHZ states, whose circuit is built in Section 1.

7. **IBM Quantum** (2024). *Qiskit documentation and learning platform*. [https://docs.quantum.ibm.com/](https://docs.quantum.ibm.com/) — Official Qiskit documentation including the Primitives V2 API (`StatevectorEstimator`, `Sampler`) used in this notebook.

8. **Schuld, M. & Petruccione, F.** (2021). *Machine Learning with Quantum Computers*. Springer. [https://doi.org/10.1007/978-3-030-83098-4](https://doi.org/10.1007/978-3-030-83098-4) — Graduate-level textbook connecting PennyLane-style differentiable quantum programming to quantum machine learning; highly recommended for Exercises 1 and 2.

---

<!-- 
SEO TAGS / KEYWORDS — for repository discoverability:
qiskit pennylane comparison
quantum computing tutorial
bell state tutorial
GHZ state quantum
parameter shift rule
quantum gradient descent
variational quantum eigensolver
quantum machine learning
QNode pennylane
QuantumCircuit qiskit
automatic differentiation quantum
quantum optimisation python
Quantum Computing Education Series by Trinexis
qiskit estimator primitives
pennylane qml.grad
quantum computing jupyter notebook
entanglement tutorial
quantum software frameworks
IBM quantum python
quantum circuit python
differentiable quantum programming
quantum expectation value
quantum computing education
CHSH Bell inequality
quantum computing beginner
-->

---

## Related Notebooks in This Series

| # | Notebook | Topic |
|---|---|---|
| Demo 1–2 | `Demo1-2_Double_Slit_and_Stern_Gerlach.ipynb` | Wave-particle duality, Stern–Gerlach experiment |
| Demo 3 | `Demo3_QMPostulates_BraKet_Bloch.ipynb` | QM postulates, Dirac notation, Bloch sphere |
| Demo 4 | `Demo4_BlochSphere_DensityMatrix.ipynb` | Density matrix formalism, mixed states |
| Demo 5 | `Demo5_Purity_Coherence_Entanglement.ipynb` | Purity, coherence, entanglement measures |
| Demo 6 | `Demo6_Noise_and_Information_Measures.ipynb` | Quantum noise, von Neumann entropy |
| Demo 7 | `Demo7_Quantum_Gates_Demo.ipynb` | Single- and multi-qubit quantum gates |
| Demo 8 | `Demo8_QuantumCircuits_EntanglingGates_WHT.ipynb` | Quantum circuits, entangling gates, Walsh–Hadamard transform |
| Demo 9 | `Demo9_Qiskit_Introduction.ipynb` | **Qiskit** — circuits, primitives, transpiler, IBM hardware |
| Demo 10 | `Demo10_PennyLane_Introduction_Hands_On.ipynb` | **PennyLane** — QNode, autodiff, variational training |
| **Demo 11** | **`Demo11_Qiskit_PennyLane_Synthesis_and_Comparison.ipynb`** | **This notebook — side-by-side comparison and capstone** |
| Demo 12b | `Demo12b_Qiskit_Oracles_Primitives_DJ.ipynb` | Oracles, Deutsch–Jozsa algorithm |
| Demo 13b | `Demo13b_Bernstein_Vazirani_Qiskit.ipynb` | Bernstein–Vazirani algorithm |
| Demo 14 | `Demo14_Simons_Algorithm_Qiskit.ipynb` | Simon's algorithm |
| Demo 15 | `Demo15_Grover_Qiskit.ipynb` | Grover's search algorithm |

---

*Prepared for the Quantum Computing Education Series by Trinexis. Runs on Google Colab — Runtime → Run all.*
