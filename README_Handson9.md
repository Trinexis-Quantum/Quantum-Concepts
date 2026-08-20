# Hands-On 9: Getting Started with Qiskit

[![Qiskit](https://img.shields.io/badge/Qiskit-2.5.0-6929C4?logo=ibm&logoColor=white)](https://www.ibm.com/quantum/qiskit)
[![Python](https://img.shields.io/badge/Python-3.9%2B-3776AB?logo=python&logoColor=white)](https://www.python.org/)
[![Jupyter](https://img.shields.io/badge/Jupyter-Notebook-F37626?logo=jupyter&logoColor=white)](https://jupyter.org/)
[![Open In Colab](https://img.shields.io/badge/Open%20In-Colab-F9AB00?logo=googlecolab&logoColor=white)](https://colab.research.google.com/)
[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)
[![Series](https://img.shields.io/badge/Series-Quantum_Computing_Education-6929C4?style=flat-square)](https://github.com/Trinexis-Quantum/Quantum-Concepts)

---

## Overview

Every craft has its tools. A carpenter who has spent weeks learning how wood responds to hand planes, chisels, and saws, understanding the grain, the tension, the physics of cutting, is then in a position to appreciate a well-designed power tool not as magic, but as systematised craft. This notebook occupies exactly that position in the Quantum Computing Education Series by Trinexis course. You have already built quantum circuits, state vectors, density matrices, noise channels, and even your own from-scratch statevector simulator using nothing but NumPy. Now you pick up **Qiskit**, IBM's open-source Python framework for quantum computing, and immediately recognise its internals, because you built the same things yourself.

Qiskit grew up as the primary interface to **real IBM Quantum hardware**: actual superconducting-qubit processors accessible over the cloud. Its design reflects that lineage at every level. A `QuantumCircuit` is an explicit ordered list of gates, not an implicit computation graph. Its two central **primitives**, `Sampler` (shot-based bitstring counts) and `Estimator` (expectation values of observables), mirror exactly what a physical experiment can return. Its **transpiler** is a full compiler that rewrites an abstract circuit into one expressible using a specific device's native gate vocabulary. Understanding these design choices, not just the syntax, is what separates a practitioner from someone who copy-pastes code.

This notebook threads through ten sections with a consistent strategy: every concept is introduced as a Qiskit expression of something you have already derived by hand. Deutsch's algorithm appears here as eight lines of Qiskit, but you built it from raw matrices in Notebook 1. Bell and GHZ states appear as three-line circuits, but you derived their entanglement properties from density matrices in Notebooks 4 and 5. The parameter-shift gradient rule is derived symbolically and then used to implement a complete miniature Variational Quantum Eigensolver (VQE) from scratch. By the end, you are not learning Qiskit syntax; you are recognising a well-designed framework for things you already understand deeply.

---

## Learning Objectives

After completing this notebook, you will be able to:

- Construct and introspect `QuantumCircuit` objects in Qiskit, and explain what the framework is actually bookkeeping on your behalf.
- Use `Statevector.from_instruction` to perform exact statevector simulation of small circuits and interpret its output in terms of the Born rule.
- Use the `StatevectorSampler` primitive to generate shot-based measurement counts and connect them to the Monte Carlo Born-rule sampling you implemented by hand.
- Implement Deutsch's algorithm end-to-end in Qiskit, and explain why measurement of the query qubit deterministically identifies the oracle type in a single query.
- Construct Bell states ($|\Phi^+\rangle$) and GHZ states for 2, 3, and 4 qubits using Hadamard and CNOT gates, and verify their entanglement structure via probability dictionaries.
- Create parameterised circuits with `Parameter` objects, bind numerical values with `assign_parameters`, and use `StatevectorEstimator` to compute expectation values of Pauli observables.
- Derive and apply the **parameter-shift rule** to compute exact analytic gradients of expectation values, without automatic differentiation, and verify the result matches $-\sin\theta$ to machine precision.
- Implement a complete gradient-descent **VQE loop** from scratch using parameter-shift gradients, and confirm convergence to the exact ground-state energy of a toy Hamiltonian.
- Explain what **transpilation** does, why it is necessary for real hardware, and identify which native gates replace an abstract Hadamard on IBM hardware.
- Describe the roles of `qiskit-aer`, `qiskit-algorithms`, and `qiskit-ibm-runtime` in the broader ecosystem, and know when each becomes relevant.
- Articulate the key architectural difference between Qiskit and PennyLane regarding automatic differentiation.

---

## Background and Theory

### What is a Quantum Circuit?

A quantum circuit is, at its mathematical core, a sequence of unitary matrices applied to a register of qubits initialised in the state $|0\cdots0\rangle$. If you have gates $U_1, U_2, \ldots, U_d$ applied in that order, the final state is:

$$|\psi_{\text{final}}\rangle = U_d \cdots U_2 \, U_1 \, |0\cdots0\rangle$$

Each gate $U_i$ typically acts on only one or two qubits, the others pass through unchanged (tensor-producted with the identity). A Qiskit `QuantumCircuit` is precisely an implementation of this bookkeeping: it tracks which gate goes on which qubit at which time step, enforces the order, and provides rendering and analysis tools around that ordered list.

### The Born Rule and the Two Things a Measurement Returns

When you measure an $n$-qubit state $|\psi\rangle = \sum_x \alpha_x |x\rangle$ in the computational basis, the probability of outcome $x$ is $p_x = |\alpha_x|^2$. This is the **Born rule**. In a real experiment, or a realistic simulation, you do not see $p_x$ directly; you see a stream of bitstrings sampled according to these probabilities. After $N$ shots you have empirical counts that converge to the true probabilities as $N \to \infty$.

Qiskit encodes this two-level reality cleanly:
- `Statevector` gives you the ideal amplitudes $\alpha_x$, useful for analysis and small-circuit verification, but not what hardware returns.
- `Sampler` gives you shot-based counts from a circuit with explicit measurement gates, this is what hardware returns.
- `Estimator` gives you $\langle\psi|O|\psi\rangle$ for a Hermitian observable $O$, directly useful for variational algorithms and chemistry.

### Deutsch's Algorithm: Interference as a Computational Resource

David Deutsch's 1985 paper introduced the first provably quantum-advantaged algorithm. The problem is simple: you are given a black-box function $f:\{0,1\}\to\{0,1\}$ that is either *constant* (same output for both inputs) or *balanced* (different outputs). Classically you need two queries to determine which. Quantumly, one suffices.

The mechanism is interference. The query qubit is placed in superposition $|+\rangle = \frac{1}{\sqrt{2}}(|0\rangle + |1\rangle)$ and the ancilla in $|-\rangle = \frac{1}{\sqrt{2}}(|0\rangle, |1\rangle)$. The oracle applies a **phase kickback**: the $f$-dependent phase imprints on the query qubit's coefficients rather than on the ancilla's state. A final Hadamard on the query qubit then causes constructive or destructive interference:

- **Constant oracle**: the two paths interfere constructively at $|0\rangle$. Measure: always 0.
- **Balanced oracle**: the two paths interfere destructively at $|0\rangle$, constructively at $|1\rangle$. Measure: always 1.

One measurement, one bit, one answer. The circuit in this notebook produces exactly $\{|0\rangle: 1000\}$ or $\{|1\rangle: 1000\}$ shots, with zero ambiguity.

### Entanglement: Bell and GHZ States

The **Bell state** $|\Phi^+\rangle = \frac{1}{\sqrt{2}}(|00\rangle + |11\rangle)$ is the two-qubit maximally entangled state. It is created by a Hadamard on qubit 0 followed by a CNOT:

$$|00\rangle \xrightarrow{H \otimes I} \frac{1}{\sqrt{2}}(|0\rangle + |1\rangle)|0\rangle \xrightarrow{\text{CNOT}} \frac{1}{\sqrt{2}}(|00\rangle + |11\rangle)$$

The **GHZ state** generalises this to $n$ qubits: $\frac{1}{\sqrt{2}}(|0\cdots0\rangle + |1\cdots1\rangle)$. It is produced by one Hadamard followed by a chain of CNOTs. Measuring any one qubit instantly constrains all others, yet no individual qubit has a definite value. This nonclassical correlation is what earlier notebooks showed cannot be reproduced by any local hidden-variable model (see the CHSH inequality, covered in a later session).

### Parameterised Circuits and Variational Algorithms

A **parameterised circuit** replaces one or more gate angles with symbolic `Parameter` placeholders. The circuit shape, the topology of gates, stays fixed while the angles vary. This is the architecture behind every **Variational Quantum Algorithm (VQA)**: a classical optimiser iteratively adjusts the parameters to minimise an objective function computed on the quantum device (or simulator). The most important VQAs are VQE (Variational Quantum Eigensolver, for ground-state chemistry) and QAOA (Quantum Approximate Optimisation Algorithm, for combinatorial problems).

### The Parameter-Shift Rule: Exact Gradients Without Autodiff

For any gate of the form $G(\theta) = e^{-i\theta P/2}$ where $P$ is a Pauli operator ($P^2 = I$), the expectation value $f(\theta) = \langle\psi|O|\psi(\theta)\rangle$ is *exactly* sinusoidal in $\theta$. This means its derivative is given by:

$$\frac{df}{d\theta} = \frac{1}{2}\left[f\!\left(\theta + \frac{\pi}{2}\right), f\!\left(\theta, \frac{\pi}{2}\right)\right]$$

This is the **parameter-shift rule**. Unlike finite differences, it is not an approximation, there is no step-size error, no numerical instability. It requires exactly two circuit evaluations per parameter per gradient step, regardless of circuit size. On real hardware, where you cannot access the circuit's internal state, this is the standard method for computing gradients. PennyLane applies it automatically under `qml.grad`; Qiskit expects you to apply it yourself (or use `qiskit-algorithms`).

### Transpilation: Circuits as Compiled Code

Every real quantum processor has a **native gate set**, a small, fixed repertoire of physically implementable operations determined by the device's hardware. IBM devices typically support `{RZ, SX, X, CX}`. An abstract gate like a Hadamard $H$ is not directly in that set; the transpiler must decompose it:

$$H = R_Z\!\left(\frac{\pi}{2}\right) \cdot \sqrt{X} \cdot R_Z\!\left(\frac{\pi}{2}\right)$$

(up to a global phase). The transpiler also maps logical qubits onto physical qubits that are actually connected on the device, inserting SWAP gates as needed. Transpilation is conceptually a full compilation pass: it is the layer that makes abstract quantum programs executable on specific physical hardware.

---

## Prerequisites

Before working through this notebook, you should be comfortable with:

**Mathematics and physics:**
- Complex numbers, vectors, and matrices (especially unitary matrices)
- The Dirac bra-ket notation $|0\rangle, |1\rangle, |\psi\rangle$
- The Born rule: $p_x = |\langle x|\psi\rangle|^2$
- Tensor products for multi-qubit systems

**Python and numerical computing:**
- NumPy arrays and basic linear algebra (`np.dot`, `np.kron`, `np.linalg.eigvalsh`)
- Matplotlib for plotting
- Basic Python: functions, loops, list comprehensions

**Prior course notebooks (strongly recommended):**
- **Demo1-2** (Double Slit, Stern-Gerlach): Born rule and measurement intuition
- **Demo3** (QM Postulates, Bra-Ket): Hilbert-space formalism
- **Demo7** (Quantum Gates): single- and two-qubit gates, especially Hadamard and CNOT
- **Demo8** (Quantum Circuits): circuit-as-matrix-product, entangling gate families

**Runtime:**
- A Google Colab account (free), or a local Python 3.9+ environment
- No Qiskit pre-installation required, the notebook installs it automatically

---

## Notebook Walkthrough

### Section 0: Setup

The notebook begins with a self-contained install cell that checks for Qiskit and `pylatexenc` (needed for the `draw('mpl')` renderer) and installs them only if missing. This pattern, import-first, install-on-demand, is a best practice for shared notebooks: it does not force a re-download on reruns and makes the notebook idempotent.

The core import block brings in `QuantumCircuit`, `transpile`, `Parameter`, `Statevector`, `SparsePauliOp`, `StatevectorSampler`, `StatevectorEstimator`, and `plot_histogram`. Notice what is *not* imported: `qiskit-aer`. The notebook deliberately uses only Qiskit's built-in reference simulator (`Statevector`) and the core primitives that ship with `qiskit` itself, keeping the install lightweight and the runtime reliable on a cold Colab machine.

### Section 1: Anatomy of a Qiskit Circuit

A one-qubit Hadamard circuit introduces the three key verbs: `QuantumCircuit(n)` allocates $n$ qubits, `.h(0)` appends a gate to the gate list, and `.draw()` renders the circuit. The text and matplotlib renderers produce the same circuit, the latter is publication-quality. This section emphasises that `qc.h(0)` is *pure bookkeeping*: Qiskit records "Hadamard on qubit 0 at this position in the sequence." No numerical computation happens until you simulate.

### Section 2: Simulating with Statevector

`Statevector.from_instruction(qc)` applies the circuit to $|0\rangle$ and returns the exact amplitude vector. For the one-qubit Hadamard, the output is $\frac{1}{\sqrt{2}}[1, 1]^T$, and `.probabilities_dict()` returns $\{|0\rangle: 0.5,\ |1\rangle: 0.5\}$. The commentary is deliberate: nothing new has happened physically. The same numbers appeared in Demo3 when you applied the $H$ matrix to a NumPy vector by hand. Qiskit is doing the same operation with maintained, tested infrastructure.

### Section 3: Measurement and the Sampler Primitive

This section introduces the crucial conceptual boundary between *exact probabilities* (what `Statevector` gives) and *samples* (what real experiments and the `Sampler` give). A measurement gate is added explicitly, `qc_meas.measure(0, 0)`, connecting qubit 0 to classical bit 0. The `StatevectorSampler` is seeded for reproducibility and run for 1000 shots. The output is empirical counts: roughly 503 zeros and 497 ones, fluctuating around the ideal 500/500 split by shot noise. The `plot_histogram` call gives the familiar bar chart you built by hand in Demo1-2. The key lesson: hardware can only return samples. `Sampler` is the faithful model of that reality.

### Section 4: Deutsch's Algorithm, Full Circle

The notebook implements the two-qubit Deutsch circuit as a named function, then runs it for both oracle types. The measurement output is deterministic: `{'0': 1000}` for the constant oracle, `{'1': 1000}` for the balanced one. The circuit diagram shows the structure: $X$ and $H$ on the ancilla to prepare $|-\rangle$, $H$ on the query qubit, the optional CNOT oracle, then $H$ on the query qubit before measurement. Students who built this from raw NumPy matrices in Demo1-2 will immediately recognise the structure and understand *why* every shot gives the same answer: the interference is perfectly constructive or destructive, leaving no probability for the wrong outcome.

### Section 5: Bell and GHZ States

Three circuits in rapid succession, one for Bell, one for GHZ-3, produce `probabilities_dict()` outputs of `{'00': 0.5, '11': 0.5}` and `{'000': 0.5, '111': 0.5}`. The text notes what is significant: $|01\rangle$, $|10\rangle$, and all mixed-parity GHZ terms have probability exactly zero. Measuring one qubit collapses the entire joint state. The code is intentionally minimal, three lines per circuit, to show that the conceptual heaviness is already behind you; the Qiskit expression is just notation.

### Section 6: Parameterised Circuits

`Parameter('theta')` creates a symbolic angle that can be embedded in a gate: `qc_param.rx(theta, 0)`. The circuit stores a placeholder; `.draw()` shows `Rx(theta)` textually. `assign_parameters({theta: np.pi/3})` binds the value, producing an ordinary runnable circuit. The resulting statevector, $[\cos(\pi/6),\ -i\sin(\pi/6)]^T \approx [0.866, -0.5i]^T$, matches the analytic $RX(\theta)|0\rangle$ formula exactly. This section establishes the parameterised circuit as a function of angles, the function that variational algorithms optimise.

### Section 7: The Estimator Primitive and Expectation Values

`SparsePauliOp.from_list([("Z", 1.0)])` constructs the Pauli-$Z$ observable. `StatevectorEstimator.run([(bound_circuit, observable)])` returns a floating-point expectation value directly, without sampling. The notebook computes $\langle Z\rangle$ as a function of $\theta$ over a full $2\pi$ sweep and plots it against the analytic $\cos\theta$. The curves are indistinguishable, they overlay exactly. An ipywidgets slider (Colab-native) lets students interactively explore how $\langle Z\rangle$ tracks the Bloch-sphere latitude as $\theta$ changes. The geometric interpretation is explicit: $RX(\theta)|0\rangle$ rotates the Bloch vector from the north pole along a meridian, and $\langle Z\rangle$ measures how far toward the equator it has gone.

### Section 8: The Parameter-Shift Rule, No Autodiff Needed

This is the conceptual centrepiece. The notebook first names the architectural difference from PennyLane: Qiskit gives you values, not gradients. Then it derives and implements the parameter-shift formula in five lines of Python:

```python
def grad_paramshift(theta_val):
    return (expZ(theta_val + shift) - expZ(theta_val - shift)) / 2
```

The shift is $\pi/2$. The notebook evaluates this at $\theta = \pi/4$, compares it to the analytic $-\sin(\pi/4) = -1/\sqrt{2}$, and shows the difference is `0.00e+00`, identically zero, not just small. This is the exact gradient, not a finite-difference approximation. The accompanying commentary explains why: because $f(\theta)$ is sinusoidal with known frequency 1, its derivative at any point is pinned by values at $\pm\pi/2$ away, a fact that holds algebraically rather than asymptotically.

### Section 9: A Miniature VQE from Scratch

The notebook builds a complete variational quantum eigensolver loop without using any library beyond core Qiskit. A two-qubit ansatz, $RY(t_1)$ on qubit 0, $RY(t_2)$ on qubit 1, followed by a CNOT, is parameterised by `t1` and `t2`. The Hamiltonian is $H = Z \otimes Z$ with known exact ground energy $-1$. The gradient-descent loop:

1. Evaluates `energy(params)` via the `Estimator`
2. Computes `grad_energy(params)` via parameter-shift (two circuits per parameter, four total per step)
3. Updates `params -= lr * grad`
4. Repeats for 60 steps

The energy history converges smoothly to $-1.000000$, matching `np.linalg.eigvalsh(H_zz.to_matrix()).min()` exactly. The convergence plot visually confirms the variational principle: the energy decreases monotonically and saturates at the exact eigenvalue. This is a complete, runnable, pedagogically transparent VQE.

### Section 10: Transpilation

A two-qubit Bell-state circuit is transpiled using `transpile(qc, basis_gates=['rz', 'sx', 'x', 'cx'], optimization_level=1)`. Before transpilation: `{'h': 1, 'cx': 1}`. After: `{'rz': 2, 'sx': 1, 'cx': 1}`. The Hadamard has been decomposed into $R_Z$ and $\sqrt{X}$, exactly the two gates IBM hardware can physically implement. The CNOT survives unchanged because it is already native. The transpiled circuit diagram shows the longer but hardware-compatible gate sequence. The commentary draws the parallel to classical compilation: same semantics, different instruction set.

### Section 11: The Wider Ecosystem

This section is a guided tour rather than executable code. Three packages are introduced:

- **qiskit-aer**: high-performance simulator with configurable noise models (depolarising, thermal relaxation, readout error). Required when circuits are too large for exact statevector simulation or when you want to model a specific device's noise profile.
- **qiskit-algorithms**: pre-built `VQE`, `QAOA`, and classical optimiser classes. Having built the loop by hand in Section 9, students can now appreciate these as exactly what they wrote, packaged and tested.
- **qiskit-ibm-runtime**: client for real IBM Quantum hardware. The same `Sampler`/`Estimator` primitives, but backed by a superconducting-qubit processor. An illustrative (non-executable) code block shows the four lines that would swap a simulator for real hardware access.

### Section 12: Graded Exercises

Three exercises of escalating difficulty:

**Exercise 1 (one star), GHZ-4:** Extend the GHZ construction to four qubits by chaining three CNOT gates: `cx(0,1)`, `cx(1,2)`, `cx(2,3)`. The scaffold runs without error but outputs the wrong probabilities (it creates `{'0000': 0.5, '0001': 0.5}` rather than `{'0000': 0.5, '1111': 0.5}`), so students know immediately when they have succeeded.

**Exercise 2 (two stars), XX Hamiltonian:** Replace $H = Z \otimes Z$ with $H = X \otimes X$ in the VQE loop. The same ansatz applies; the gradient-descent loop is the same. The challenge is recognising that the Estimator call needs to reference the new Hamiltonian, and that the ansatz can still reach the ground state (eigenvalue $-1$) of this operator even though it is diagonal in a different basis.

**Exercise 3 (three stars), Deeper Ansatz:** A four-term Hamiltonian $H = X\otimes X + 0.7\,(Z\otimes I) + 0.4\,(I\otimes X) + 0.3\,(Z\otimes Z)$ with exact ground energy approximately $-1.721$ is presented. The two-parameter shallow ansatz from Section 9 can only reach approximately $-1.604$, it lacks the expressibility to span the true ground state. Students must construct a two-layer ansatz (four parameters, two CNOT layers), apply the parameter-shift gradient over all four parameters, and confirm that the deeper circuit gets substantially closer to the exact answer. This exercise teaches the concept of **ansatz expressibility** and the trade-off between circuit depth and optimisation accuracy.

---

## Key Takeaways

- A Qiskit `QuantumCircuit` is an ordered gate list, not a computation graph. Simulation, sampling, and gradient computation are all explicit, separate operations, which makes the framework transparent and hardware-honest.
- `Statevector` (exact amplitudes) and `Sampler` (shot-based counts) represent the two fundamentally different things you can know about a quantum state: the ideal mathematical description vs. what any physical experiment actually returns.
- Deutsch's algorithm demonstrates that quantum computation gains its power from interference, not parallelism: the two oracle paths are both "evaluated" simultaneously, but the interference of their contributions is what produces a single deterministic answer.
- The parameter-shift rule gives **exact** analytic gradients of quantum circuits using only two forward evaluations per parameter, no automatic differentiation, no finite differences, no approximation.
- A complete VQE loop requires fewer than 30 lines of code: a parameterised ansatz, an `Estimator`-based energy function, parameter-shift gradients, and a gradient-descent update. The core idea is that simple.
- **Ansatz expressibility** matters: a circuit with too few parameters cannot, in principle, represent the true ground state, regardless of how long you optimise it. Deeper circuits expand the reachable manifold of states.
- Transpilation is quantum compilation. The abstract circuit you write and the machine-runnable circuit are related by a compiler that knows the device's native gates, qubit connectivity, and calibration, exactly as a classical compiler knows a CPU's instruction set.

---

## Further Reading and Citations

1. **Nielsen, M. A., & Chuang, I. L. (2010).** *Quantum Computation and Quantum Information* (10th Anniversary Edition). Cambridge University Press., The standard graduate reference. Chapter 1 introduces quantum circuits; Chapter 4 covers quantum algorithms including Deutsch's; Chapter 5 covers the quantum Fourier transform and its applications. ISBN: 978-1-107-00217-3.

2. **Deutsch, D. (1985).** Quantum theory, the Church–Turing principle and the universal quantum computer. *Proceedings of the Royal Society of London. Series A, Mathematical and Physical Sciences*, 400(1818), 97–117. [doi:10.1098/rspa.1985.0070](https://doi.org/10.1098/rspa.1985.0070), The original paper introducing the Deutsch algorithm and the concept of a universal quantum computer.

3. **Peruzzo, A., McClean, J., Shadbolt, P., Yung, M.-H., Zhou, X.-Q., Love, P. J., Aspuru-Guzik, A., & O'Brien, J. L. (2014).** A variational eigenvalue solver on a photonic chip. *Nature Communications*, 5, 4213. [doi:10.1038/ncomms5213](https://doi.org/10.1038/ncomms5213), The original VQE paper: demonstrates the variational quantum eigensolver on a photonic chip, pioneering the idea of hybrid classical-quantum optimisation for chemistry.

4. **Mitarai, K., Negoro, M., Kitagawa, M., & Fujii, K. (2018).** Quantum circuit learning. *Physical Review A*, 98(3), 032309. [arXiv:1803.00745](https://arxiv.org/abs/1803.00745), Introduces the parameter-shift rule in the context of learning quantum circuit functions, showing how to compute exact gradients of parametric quantum circuits using hardware-executable circuits.

5. **Schuld, M., Bergholm, V., Gogolin, C., Izaac, J., & Killoran, N. (2019).** Evaluating analytic gradients on quantum hardware. *Physical Review A*, 99(3), 032331. [arXiv:1811.11184](https://arxiv.org/abs/1811.11184), Rigorous derivation and generalisation of the parameter-shift rule for arbitrary Pauli-generator gates, establishing its exactness and its applicability to real hardware.

6. **Cerezo, M., Arrasmith, A., Babbush, R., Benjamin, S. C., Endo, S., Fujii, K., ... & Coles, P. J. (2021).** Variational quantum algorithms. *Nature Reviews Physics*, 3, 625–644. [arXiv:2012.09265](https://arxiv.org/abs/2012.09265), Comprehensive review of VQAs including VQE, QAOA, and quantum machine learning; covers expressibility, barren plateaus, and hardware considerations. Essential reading for anyone pursuing variational methods.

7. **IBM Quantum & Qiskit Community. (2024).** *Qiskit Documentation* (v2.x). IBM. [https://docs.quantum.ibm.com/](https://docs.quantum.ibm.com/), Official documentation covering `QuantumCircuit`, primitives, transpilation, and the full Qiskit ecosystem. Continuously updated to track the latest SDK versions.

8. **Preskill, J. (1998–2023).** *Lecture Notes for Physics 219/Computer Science 219: Quantum Computation*. California Institute of Technology. [http://theory.caltech.edu/~preskill/ph229/](http://theory.caltech.edu/~preskill/ph229/), Freely available lecture notes covering quantum information theory, quantum algorithms, and quantum error correction at a graduate level. Chapter 6 is particularly relevant for quantum circuits and algorithms.

---

<!-- 
SEO TAGS / KEYWORDS:
qiskit tutorial
qiskit introduction
IBM quantum computing
quantum circuit python
statevector simulation
quantum sampler primitive
quantum estimator primitive
Deutsch algorithm qiskit
Bell state qiskit
GHZ state quantum circuit
parameterized quantum circuit
parameter shift rule
variational quantum eigensolver
VQE tutorial
quantum gradient descent
quantum transpilation
qiskit-ibm-runtime
Quantum Computing Education Series by Trinexis
hands-on quantum computing
quantum machine learning
qiskit 2.5
SparsePauliOp
quantum expectation value
quantum computing course
IBM quantum hardware
qiskit aer
quantum algorithms python
entanglement quantum circuit
quantum superposition circuit
jupyter quantum notebook
-->

---

## Related Notebooks in This Series

This notebook is **Hands-On 9** of the Quantum Computing Education Series by Trinexis (Quantum Computing Education Series by Trinexis), 2026. The full series builds from physical intuition to production frameworks:

| # | Notebook | Key Topics |
|---|----------|------------|
| 1-2 | `Demo1-2_Double_Slit_and_Stern_Gerlach.ipynb` | Wave-particle duality, Born rule, Stern-Gerlach experiment, measurement as sampling |
| 3 | `Demo3_QMPostulates_BraKet_Bloch.ipynb` | Dirac notation, postulates of quantum mechanics, Bloch sphere |
| 4 | `Demo4_BlochSphere_DensityMatrix.ipynb` | Density matrices, mixed states, partial trace |
| 5 | `Demo5_Purity_Coherence_Entanglement.ipynb` | Purity, coherence measures, entanglement quantification |
| 6 | `Demo6_Noise_and_Information_Measures.ipynb` | Quantum noise channels, depolarising and dephasing noise, quantum entropy |
| 7 | `Demo7_Quantum_Gates_Demo.ipynb` | Single- and multi-qubit gates, matrix representations, gate decompositions |
| 8 | `Demo8_QuantumCircuits_EntanglingGates_WHT.ipynb` | Circuit-as-matrix-product, from-scratch statevector simulator, Walsh-Hadamard transform |
| **9** | **`Demo9_Qiskit_Introduction.ipynb`** (this notebook) | **Qiskit framework, Sampler/Estimator, Deutsch, VQE, transpilation** |
| 10 | `Demo10_PennyLane_Introduction_Hands_On.ipynb` | PennyLane framework, automatic differentiation, QNodes |
| 11 | `Demo11_Qiskit_PennyLane_Synthesis_and_Comparison.ipynb` | Side-by-side VQE comparison: Qiskit vs. PennyLane, same physics, different mechanics |
| 12b | `Demo12b_Qiskit_Oracles_Primitives_DJ.ipynb` | Deutsch-Jozsa algorithm, oracle construction, Qiskit primitives |
| 13b | `Demo13b_Bernstein_Vazirani_Qiskit.ipynb` | Bernstein-Vazirani algorithm, hidden bitstring, linear quantum queries |
| 14 | `Demo14_Simons_Algorithm_Qiskit.ipynb` | Simon's algorithm, hidden period finding, exponential quantum speedup |
| 15 | `Demo15_Grover_Qiskit.ipynb` | Grover's search algorithm, amplitude amplification, quadratic speedup |

---

*Prepared for the Quantum Computing Education Series by Trinexis, 2026. Runs fully on Google Colab, Runtime → Run all.*
