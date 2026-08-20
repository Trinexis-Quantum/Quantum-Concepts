<!-- SEO META
quantum computing, qiskit, pennylane, quantum algorithms, quantum gates, grover algorithm, simon algorithm, bernstein vazirani, deutsch jozsa, quantum circuits, bloch sphere, density matrix, quantum entanglement, quantum noise, quantum information, quantum hackathon, quantum challenge, hands-on quantum computing, quantum programming, quantum computing tutorial, quantum computing for beginners, quantum computing python, IBM quantum, quantum machine learning, quantum computing education, quantum computing, quantum computing course, quantum computing research, qiskit tutorial, pennylane tutorial, quantum superposition, quantum interference, stern gerlach, double slit experiment, bra-ket notation, quantum postulates, purity coherence, Kraus operators, quantum error correction, NISQ, variational quantum algorithms
-->

<div align="center">

# ⚛️ Quantum Concepts - Hands-On Lab Series

### *From First Principles to Quantum Algorithms: A Practitioner's Journey*

[![Python](https://img.shields.io/badge/Python-3.9%2B-3776AB?style=for-the-badge&logo=python&logoColor=white)](https://www.python.org/)
[![Qiskit](https://img.shields.io/badge/Qiskit-2.5.0-6929C4?style=for-the-badge&logo=ibm&logoColor=white)](https://qiskit.org/)
[![PennyLane](https://img.shields.io/badge/PennyLane-0.45-00B0F0?style=for-the-badge&logo=xrp&logoColor=white)](https://pennylane.ai/)
[![Jupyter](https://img.shields.io/badge/Jupyter-Notebook-F37626?style=for-the-badge&logo=jupyter&logoColor=white)](https://jupyter.org/)
[![License: MIT](https://img.shields.io/badge/License-MIT-green?style=for-the-badge)](LICENSE)
[![Stars](https://img.shields.io/github/stars/Trinexis-Quantum/Quantum-Concepts?style=for-the-badge&color=yellow)](https://github.com/Trinexis-Quantum/Quantum-Concepts/stargazers)

**Quantum Computing Education Series by Trinexis**

[📚 Start Learning](#-learning-path--roadmap) · [🗺️ Roadmap](#-learning-path--roadmap) · [🏆 Hackathons](#-quantum-hackathons-challenges--projects) · [📖 Notebooks](#-notebook-series--what-youll-learn) · [⚙️ Setup](#-getting-started)

</div>

---

## 🌌 Why Quantum Computing - and Why Now?

We are living through a technological inflection point. Quantum computers - machines that harness superposition, entanglement, and interference - are transitioning from laboratory curiosity to practical tools capable of solving problems that would take classical supercomputers longer than the age of the universe. IBM, Google, IonQ, and dozens of startups are racing to build fault-tolerant quantum hardware. Governments worldwide are investing billions. **The question is no longer *if* quantum computing will matter - it's whether *you* will be ready when it does.**

This repository is your hands-on entry point. Across **15 carefully structured Jupyter notebooks**, you will travel from the strange, counterintuitive physics of the quantum world - double-slit experiments, spinning electrons, wave-particle duality - all the way through to implementing real quantum algorithms in **Qiskit** and **PennyLane**, the two dominant quantum programming frameworks. Every concept is paired with runnable code so you can *see* the quantum world, not just read about it.

Whether you are a **student** taking your first quantum course, a **researcher** adding quantum tools to your repertoire, or an **engineer** preparing for quantum hackathons and industry challenges, this series is designed to grow with you. No prior quantum knowledge is assumed - only curiosity, a Python environment, and the willingness to have your classical intuitions pleasantly shattered.

---

## 📦 Repository at a Glance

| | |
|---|---|
| **Notebooks** | 15 hands-on Jupyter labs |
| **Frameworks** | Qiskit 2.5.0 · PennyLane 0.45 · NumPy · Matplotlib |
| **Language** | Python 3.9+ |
| **Level** | Beginner → Intermediate → Advanced |
| **Topics** | QM Foundations · Quantum Gates · Circuits · Algorithms · Simulators |
| **License** | MIT |

---

## 🗺️ Learning Path & Roadmap

The notebooks follow a deliberate three-act structure. Think of it as a **quantum apprenticeship**:

```
╔══════════════════════════════════════════════════════════════════╗
║  ACT 1 - THE PHYSICS (Notebooks 1–6)                            ║
║  "Understand the rules of the game before you play"             ║
║  Quantum mechanics foundations, pure and mixed states,          ║
║  entanglement, noise, information theory                        ║
╠══════════════════════════════════════════════════════════════════╣
║  ACT 2 - THE TOOLS (Notebooks 7–11)                             ║
║  "Learn to speak the language of quantum machines"              ║
║  Quantum gates, circuits, Qiskit, PennyLane, simulators         ║
╠══════════════════════════════════════════════════════════════════╣
║  ACT 3 - THE ALGORITHMS (Notebooks 12–15)                       ║
║  "Build things that have real quantum advantage"                ║
║  Oracles, Deutsch-Jozsa, Bernstein-Vazirani, Simon's,           ║
║  Grover's search - the canonical quantum speedups               ║
╚══════════════════════════════════════════════════════════════════╝
```

Complete the acts in order. Each notebook builds on the last. By the end you will have a coherent mental model that connects the physics to the math to the code - the trifecta that separates practitioners from onlookers.

---

## 📖 Notebook Series - What You'll Learn

### 🔬 ACT 1 - Quantum Physics Foundations

---

#### [Hands-On 1 & 2 - Double Slit Experiment & Stern-Gerlach](Handson1&2_Double_Slit_and_Stern_Gerlach.ipynb) · [`README`](README_Handson1_2.md)

> *"Before you can program a qubit, you must believe it exists."*

The double-slit experiment is arguably the most profound experiment in all of science. A single electron, fired at a barrier with two slits, produces an interference pattern - as if it passed through both slits simultaneously. This is not a trick or an approximation; it is the raw, undeniable signature of quantum superposition.

The Stern-Gerlach experiment takes this further: passing silver atoms through an inhomogeneous magnetic field reveals that spin - an intrinsic quantum property with no classical analogue - is fundamentally discrete and directional.

**What you'll learn:**
- Why classical probability cannot explain quantum interference
- The Born rule: how quantum amplitudes become measurable probabilities
- Spin quantisation and why it matters for qubit design
- Simulating both experiments from scratch with NumPy and Matplotlib

**How this helps your journey:** Every qubit in every quantum computer is a physical realisation of the spin-½ system studied here. You are not just doing history - you are meeting the hardware.

---

#### [Hands-On 3 - QM Postulates, Bra-Ket & Bloch Sphere](Handson3_QMPostulates_BraKet_Bloch.ipynb) · [`README`](README_Handson3.md)

> *"Dirac's bra-ket notation is the lingua franca of quantum computing. Learn it once; use it everywhere."*

Quantum mechanics rests on five postulates. This notebook makes them concrete and computational. You will learn Dirac's elegant bra-ket notation - the language used in every research paper, every textbook, and every quantum programming framework - and visualise the state of a qubit on the Bloch sphere.

**What you'll learn:**
- The five postulates of quantum mechanics, coded and visualised
- How |0⟩ and |1⟩ are just two points on a sphere - and every superposition lives between them
- Measurement, collapse, and why you can never observe a quantum state without disturbing it
- Inner products, outer products, and expectation values by hand

**How this helps your journey:** The Bloch sphere is the mental model that will guide every gate operation you perform from here on. Rotations on this sphere *are* quantum gates.

---

#### [Hands-On 4 - Bloch Sphere & Density Matrix](Handson4_BlochSphere_DensityMatrix.ipynb) · [`README`](README_Handson4.md)

> *"Pure states are the ideal. Density matrices are the reality."*

Real quantum systems are never perfectly isolated. When a qubit interacts with its environment, it becomes entangled with countless uncontrollable degrees of freedom, and you can no longer describe it with a single state vector. Enter the **density matrix** - the most general description of a quantum state, able to capture both pure and mixed states in a single mathematical object.

**What you'll learn:**
- Constructing and interpreting density matrices
- The difference between a pure state (a perfect qubit) and a mixed state (a noisy one)
- Von Neumann entropy as a measure of quantum uncertainty
- Visualising state evolution directly on the Bloch sphere

**How this helps your journey:** Every real quantum computation on NISQ hardware involves mixed states. Understanding density matrices is prerequisite knowledge for quantum error mitigation and variational quantum algorithms.

---

#### [Hands-On 5 - Purity, Coherence & Entanglement](Handson5_Purity_Coherence_Entanglement.ipynb) · [`README`](README_Handson5.md)

> *"Entanglement is the resource that makes quantum computers powerful. This is where you learn to measure it."*

Quantum coherence is what makes superposition possible. Entanglement is what makes quantum computers exponentially more powerful than classical ones. In this notebook you will quantify both - computing purity, coherence measures, entanglement entropy, and concurrence for real quantum states.

**What you'll learn:**
- Purity: how to measure how "quantum" a state is
- Coherence measures: l₁-norm and relative entropy of coherence
- Bell states: the four maximally entangled two-qubit states
- Partial trace and reduced density matrices for multi-qubit systems
- Entanglement entropy and concurrence as entanglement quantifiers

**How this helps your journey:** Entanglement is the fuel of quantum advantage. Grover's algorithm, Shor's algorithm, quantum teleportation, and superdense coding all rely on it. You are learning to handle the fuel.

---

#### [Hands-On 6 - Noise & Quantum Information Measures](Handson6_Noise_and_Information_Measures.ipynb) · [`README`](README_Handson6.md)

> *"The enemy of every quantum computation is noise. Know your enemy."*

NISQ (Noisy Intermediate-Scale Quantum) computers are called *noisy* for a reason. Decoherence, gate errors, and measurement noise corrupt quantum states continuously. This notebook builds a complete theoretical toolkit for understanding, modelling, and quantifying noise, and connects it to the ten most important quantum information measures.

**What you'll learn:**
- Quantum channels: Kraus operators and the CPTP formalism
- The five canonical noise channels: bit flip, phase flip, depolarising, amplitude damping, phase damping
- T₁ (relaxation) and T₂ (dephasing) times: the fundamental hardware metrics
- Ten information measures: von Neumann entropy, relative entropy, mutual information, Holevo bound, and more

**How this helps your journey:** Before you can run a quantum algorithm on real hardware, you need to understand noise. This notebook is the foundation for quantum error correction, error mitigation, and noisy simulation - critical skills for any quantum project.

---

### ⚙️ ACT 2 - Quantum Programming Tools

---

#### [Hands-On 7 - Quantum Gates Deep Dive](Handson7_Quantum_Gates_Demo.ipynb) · [`README`](README_Handson7.md)

> *"Quantum gates are rotations. Once you see that, everything clicks."*

Classical computers use AND, OR, and NOT gates to transform bits. Quantum computers use unitary matrices to rotate qubits on the Bloch sphere. This notebook gives you an exhaustive, hands-on tour of every important quantum gate - from Pauli operators to the universally expressive set of single-qubit rotations and two-qubit entanglers.

**What you'll learn:**
- Single-qubit gates: X (bit flip), Y, Z (phase flip), H (Hadamard), S, T, Rx, Ry, Rz
- Two-qubit gates: CNOT, CZ, SWAP, controlled-U
- Three-qubit gates: Toffoli (CCNOT), Fredkin (CSWAP)
- Gate decomposition and universality: why {H, T, CNOT} can approximate any quantum computation
- The no-cloning theorem: why you can't copy a qubit

**How this helps your journey:** Every quantum circuit - from a 3-qubit toy example to a 100-qubit Grover search - is built from these gates. This is your vocabulary lesson.

---

#### [Hands-On 8 - Quantum Circuits, Entangling Gates & WHT](Handson8_QuantumCircuits_EntanglingGates_WHT.ipynb) · [`README`](README_Handson8.md)

> *"A quantum circuit is a recipe. This notebook teaches you to read and write those recipes."*

Individual gates are words. Quantum circuits are sentences. This notebook moves from single-gate operations to composing multi-qubit circuits, building Bell states, GHZ states, and exploring the Walsh-Hadamard Transform - a quantum operation that plays a central role in many algorithms.

**What you'll learn:**
- Building and running multi-qubit circuits from first principles
- Creating Bell states and GHZ states with CNOT+H
- The Walsh-Hadamard Transform and why it creates uniform superpositions
- Circuit depth, gate count, and why these matter for NISQ hardware

**How this helps your journey:** Every algorithm in Act 3 is expressed as a circuit. This notebook is where you develop the circuit-building intuition you will rely on constantly.

---

#### [Hands-On 9 - Qiskit Introduction](Handson9_Qiskit_Introduction.ipynb) · [`README`](README_Handson9.md)

> *"Qiskit is IBM's open-source framework for quantum computing. This is your orientation session."*

Qiskit is the world's most widely used quantum programming framework, with over 550,000 registered users. This notebook is your complete onboarding: from installing Qiskit and creating your first QuantumCircuit, to visualising circuits, running statevector simulations, and interpreting measurement histograms.

**What you'll learn:**
- Qiskit architecture: Terra, circuits, primitives, and transpiler
- Creating QuantumCircuits, adding gates, measuring qubits
- Running simulations with AerSimulator and StatevectorSimulator
- Reading and interpreting measurement probability distributions
- Transpiling circuits for real IBM Quantum backends

**How this helps your journey:** Qiskit is the entry point to IBM Quantum's real hardware - the most accessible quantum computers in the world. This notebook puts that access in your hands.

---

#### [Hands-On 10 - PennyLane Introduction](Handson10_PennyLane_Introduction_Hands_On.ipynb) · [`README`](README_Handson10.md)

> *"PennyLane bridges quantum circuits and machine learning. It is the framework for the quantum AI era."*

PennyLane by Xanadu takes a different approach to quantum programming: it treats quantum circuits as differentiable functions, allowing gradients to flow through them just like neural networks. This makes it the natural framework for Quantum Machine Learning (QML) and Variational Quantum Algorithms (VQAs).

**What you'll learn:**
- PennyLane's QNode: turning quantum circuits into differentiable functions
- Devices: simulators (default.qubit) and how to target real hardware
- The parameter-shift rule: computing gradients of quantum circuits
- Building variational circuits (ansätze) and cost functions
- Your first variational quantum eigensolver (VQE) sketch

**How this helps your journey:** Most quantum hackathon problems today involve QML or VQAs. PennyLane is the dominant tool for both. This notebook is your competitive edge.

---

#### [Hands-On 11 - Qiskit vs PennyLane: Synthesis & Comparison](Handson11_Qiskit_PennyLane_Synthesis_and_Comparison.ipynb) · [`README`](README_Handson11.md)

> *"Knowing both frameworks means you can pick the right tool for every problem."*

Qiskit and PennyLane are complementary, not competing. This notebook runs the same quantum algorithms in both frameworks side-by-side, building your ability to read, write, and translate between them - a crucial skill for collaborating across the quantum ecosystem.

**What you'll learn:**
- Side-by-side implementation of Bell state preparation, quantum teleportation, and variational circuits
- Performance comparison: noise models, circuit depths, simulation fidelity
- When to choose Qiskit (gate-level control, IBM hardware) vs PennyLane (gradient-based optimisation, QML)
- Converting circuits between frameworks using PennyLane's Qiskit plugin

**How this helps your journey:** Real quantum projects rarely use just one framework. This notebook makes you bilingual in the two most important quantum languages.

---

### 🏆 ACT 3 - Quantum Algorithms with Real Advantage

---

#### [Hands-On 12 - Oracles, Primitives & Deutsch-Jozsa](Handson12_Qiskit_Oracles_Primitives_DJ.ipynb) · [`README`](README_Handson12b.md)

> *"The Deutsch-Jozsa algorithm was the first proof that quantum computers can outperform classical ones. Exponential speedup, guaranteed."*

Quantum oracles are black boxes that encode a function into a quantum circuit via phase kickback. The Deutsch-Jozsa algorithm uses a single oracle query - where a classical computer would need up to 2^(n-1)+1 - to determine whether a function is constant or balanced. It is a clean, elegant demonstration of quantum parallelism.

**What you'll learn:**
- Oracle construction: encoding Boolean functions as unitary operators
- Phase kickback: the mechanism behind most quantum speedups
- Qiskit Primitives: Sampler and Estimator for modern quantum execution
- Deutsch-Jozsa: circuit design, proof of correctness, and Qiskit implementation
- Why this exponential speedup, while impractical, is theoretically foundational

**How this helps your journey:** Oracles appear in Grover's algorithm, Simon's algorithm, and amplitude estimation. Understanding them at this level is the key to building more complex quantum solutions.

---

#### [Hands-On 13 - Bernstein-Vazirani Algorithm](Handson13_Bernstein_Vazirani_Qiskit.ipynb) · [`README`](README_Handson13.md)

> *"One query. Any string length. The hidden secret revealed instantly - that is the Bernstein-Vazirani guarantee."*

The Bernstein-Vazirani algorithm finds a hidden binary string `s` encoded in an oracle using exactly **one** quantum query, regardless of how long `s` is. Classically, finding all n bits requires n queries. This is a clean, provable quantum speedup - not asymptotic, but exact.

**What you'll learn:**
- The Bernstein-Vazirani problem: finding a hidden dot-product string
- Why the algorithm needs exactly one query (proof by quantum parallelism)
- Implementing the oracle and circuit in Qiskit
- Running on a noisy simulator and reading the results
- The relationship between BV and the Deutsch-Jozsa algorithm

**How this helps your journey:** BV is a stepping stone to understanding Simon's algorithm and Shor's algorithm. It also appears in cryptographic settings - making it directly relevant to quantum-safe security research.

---

#### [Hands-On 14 - Simon's Algorithm](Handson14_Simons_Algorithm_Qiskit.ipynb) · [`README`](README_Handson14.md)

> *"Simon's algorithm was the inspiration for Shor's algorithm. Master this and Shor's becomes intuitive."*

Simon's algorithm solves a hidden symmetry problem exponentially faster than any classical algorithm. Given an oracle that promises f(x) = f(x⊕s) for some hidden string s, Simon's algorithm finds s using O(n) quantum queries - versus O(2^(n/2)) classically. This was the first exponential quantum speedup for a natural problem, and it directly inspired Shor's celebrated factoring algorithm.

**What you'll learn:**
- Simon's problem: the hidden XOR-period promise
- The quantum circuit: uniform superposition → oracle → Hadamard → measure
- Why interference reveals the hidden string
- Solving the GF(2) linear system to recover s
- Noisy simulation on Qiskit and fidelity analysis
- The conceptual bridge from Simon's to Shor's algorithm

**How this helps your journey:** Understanding Simon's algorithm is the intellectual prerequisite for Shor's algorithm - the quantum threat to RSA encryption. If you want to work in quantum cryptography or post-quantum security, this notebook is essential.

---

#### [Hands-On 15 - Grover's Search Algorithm](Handson15_Grover_Qiskit.ipynb) · [`README`](README_Handson15.md)

> *"Searching a billion-entry database in 31,623 steps instead of 500 million. That is Grover's quadratic speedup - and it is real."*

Grover's algorithm is the most practically relevant quantum algorithm for near-term hardware. It searches an unstructured database of N items in O(√N) queries - a quadratic speedup over classical O(N) search. For a database of a trillion entries, Grover needs ~1 million queries; classical search needs 500 billion. The algorithm uses **amplitude amplification**: iteratively rotating the quantum state towards the target answer.

**What you'll learn:**
- The oracle: marking the target item with a phase flip
- The diffusion operator (Grover diffusion): the "inversion about the mean" step
- Optimal iteration count: π/4 × √N - too few or too many iterations reduce success probability
- Full Qiskit implementation with measurement and visualisation
- Multi-target Grover and generalisations
- Applications: database search, satisfiability, optimisation

**How this helps your journey:** Grover's algorithm is a building block for dozens of quantum applications - from quantum optimisation and cryptanalysis to combinatorial search in AI. It is the most asked-about algorithm in quantum hackathons.

---

## 🏆 Quantum Hackathons, Challenges & Projects

This series was explicitly designed to prepare participants for real quantum competitions and projects. Here is how each act maps to what you will encounter:

### 🎯 Hackathon Readiness Map

| Competition Type | Essential Notebooks | What You'll Be Ready For |
|---|---|---|
| **IBM Quantum Challenge** | 9, 12, 15 | Qiskit circuits, Grover's, Oracle design |
| **QHack (Xanadu)** | 10, 11, 15 | PennyLane QML, variational circuits |
| **MIT iQuHACK** | 3–8, 12–15 | Full-stack quantum: theory + code + algorithms |
| **CDL Quantum Bootcamp** | 5, 6, 10, 11 | VQA, noise-aware circuits, QML |
| **Quantum Open Source Foundation** | All | Open-source quantum tooling |

### 🔬 Hands-On Quantum Project Ideas

After completing this series, you will have the skills to tackle:

1. **Quantum Optimisation** - Use Grover's algorithm (Notebook 15) to search combinatorial solution spaces. Extend to QAOA using PennyLane (Notebooks 10–11).

2. **Quantum Machine Learning** - Build a variational quantum classifier using PennyLane's differentiable programming (Notebooks 10–11). Train it on a real dataset.

3. **Quantum Cryptography Simulator** - Implement BB84 quantum key distribution using the gate primitives from Notebooks 7–9. Simulate eavesdropping and detection.

4. **Quantum Error Mitigation** - Apply zero-noise extrapolation and probabilistic error cancellation to the noisy simulations in Notebook 6, building on the Kraus operator formalism.

5. **Quantum Oracle Design** - Design custom oracles for search problems (Notebooks 12–14), then apply Grover's amplitude amplification for structured problem solving.

6. **Simon's → Shor's Bridge Project** - Using Notebook 14 as a foundation, implement a small-scale version of Shor's factoring algorithm in Qiskit.

### 💡 How These Concepts Connect to Industry

```
Foundations (1–6)  ──→  Quantum Hardware Teams (IBM, Google, IonQ)
                         Quantum Error Correction Research
                         Quantum Sensing & Metrology

Tools (7–11)       ──→  Quantum Software Engineering
                         QML Research (pharmaceutical, finance, logistics)
                         Quantum-Classical Hybrid Applications

Algorithms (12–15) ──→  Post-Quantum Cryptography
                         Quantum Optimisation (supply chain, portfolio)
                         Search & Database acceleration
```

---

## 🚀 Getting Started

### Prerequisites

- Python 3.9 or higher
- Basic Python programming (NumPy, Matplotlib)
- Linear algebra fundamentals (vectors, matrices, complex numbers)
- No prior quantum knowledge required

### Installation

```bash
# Clone the repository
git clone https://github.com/Trinexis-Quantum/Quantum-Concepts.git
cd Quantum-Concepts

# Create a virtual environment (recommended)
python -m venv quantum-env
source quantum-env/bin/activate  # Windows: quantum-env\Scripts\activate

# Install core dependencies
pip install qiskit==2.5.0
pip install qiskit-aer
pip install pennylane==0.45
pip install pennylane-qiskit
pip install numpy matplotlib scipy jupyter

# Launch Jupyter
jupyter notebook
```

### IBM Quantum Access (for real hardware)

```bash
pip install qiskit-ibm-runtime
```

Then set your IBM Quantum API token (get one free at [quantum.ibm.com](https://quantum.ibm.com)):

```python
from qiskit_ibm_runtime import QiskitRuntimeService
QiskitRuntimeService.save_account(channel="ibm_quantum", token="YOUR_TOKEN")
```

---

## 📋 Full Notebook Index

| # | Notebook | Topic | Key Concepts | README |
|---|---|---|---|---|
| 1&2 | [Handson1&2](Handson1&2_Double_Slit_and_Stern_Gerlach.ipynb) | Double Slit & Stern-Gerlach | Superposition, interference, spin, Born rule | [📖](README_Handson1_2.md) |
| 3 | [Handson3](Handson3_QMPostulates_BraKet_Bloch.ipynb) | QM Postulates & Bra-Ket | 5 postulates, Dirac notation, Bloch sphere | [📖](README_Handson3.md) |
| 4 | [Handson4](Handson4_BlochSphere_DensityMatrix.ipynb) | Bloch Sphere & Density Matrix | Mixed states, von Neumann entropy, purity | [📖](README_Handson4.md) |
| 5 | [Handson5](Handson5_Purity_Coherence_Entanglement.ipynb) | Purity, Coherence & Entanglement | Bell states, partial trace, concurrence | [📖](README_Handson5.md) |
| 6 | [Handson6](Handson6_Noise_and_Information_Measures.ipynb) | Noise & Information Measures | Kraus operators, T₁/T₂, Holevo bound | [📖](README_Handson6.md) |
| 7 | [Handson7](Handson7_Quantum_Gates_Demo.ipynb) | Quantum Gates | Pauli, H, CNOT, Toffoli, universality | [📖](README_Handson7.md) |
| 8 | [Handson8](Handson8_QuantumCircuits_EntanglingGates_WHT.ipynb) | Circuits & WHT | Bell/GHZ states, Walsh-Hadamard Transform | [📖](README_Handson8.md) |
| 9 | [Handson9](Handson9_Qiskit_Introduction.ipynb) | Qiskit Introduction | QuantumCircuit, AerSimulator, transpiler | [📖](README_Handson9.md) |
| 10 | [Handson10](Handson10_PennyLane_Introduction_Hands_On.ipynb) | PennyLane Introduction | QNode, parameter-shift rule, VQE sketch | [📖](README_Handson10.md) |
| 11 | [Handson11](Handson11_Qiskit_PennyLane_Synthesis_and_Comparison.ipynb) | Qiskit vs PennyLane | Framework comparison, circuit conversion | [📖](README_Handson11.md) |
| 12 | [Handson12](Handson12_Qiskit_Oracles_Primitives_DJ.ipynb) | Oracles & Deutsch-Jozsa | Phase kickback, oracles, Qiskit Primitives | [📖](README_Handson12b.md) |
| 13 | [Handson13](Handson13_Bernstein_Vazirani_Qiskit.ipynb) | Bernstein-Vazirani | Hidden string, quantum parallelism | [📖](README_Handson13.md) |
| 14 | [Handson14](Handson14_Simons_Algorithm_Qiskit.ipynb) | Simon's Algorithm | XOR periodicity, GF(2) linear algebra | [📖](README_Handson14.md) |
| 15 | [Handson15](Handson15_Grover_Qiskit.ipynb) | Grover's Algorithm | Amplitude amplification, oracle, diffusion | [📖](README_Handson15.md) |

---

## 📚 Key Quantum Concepts Covered

<details>
<summary><b>🔭 Quantum Mechanics Foundations</b></summary>

- Wave-particle duality and the double-slit experiment
- Quantum superposition and the Born rule
- Spin quantisation and the Stern-Gerlach experiment
- The five postulates of quantum mechanics
- Dirac bra-ket notation
- Measurement and state collapse

</details>

<details>
<summary><b>📐 State Representation & Information</b></summary>

- State vectors and the Hilbert space
- The Bloch sphere representation
- Density matrices and mixed states
- Purity and von Neumann entropy
- Quantum coherence measures
- Entanglement: Bell states, GHZ states, entanglement entropy
- Quantum information measures: mutual information, relative entropy, Holevo bound

</details>

<details>
<summary><b>📡 Quantum Noise & Open Systems</b></summary>

- Quantum channels and the CPTP formalism
- Kraus operator representation
- Decoherence channels: bit flip, phase flip, depolarising, amplitude damping, phase damping
- T₁ relaxation and T₂ dephasing times
- NISQ hardware noise models

</details>

<details>
<summary><b>⚙️ Quantum Gates & Circuits</b></summary>

- Single-qubit gates: X, Y, Z, H, S, T, Rx, Ry, Rz
- Two-qubit gates: CNOT, CZ, SWAP, iSWAP
- Three-qubit gates: Toffoli, Fredkin
- Universal gate sets
- The no-cloning theorem
- Circuit depth and gate complexity
- Walsh-Hadamard Transform

</details>

<details>
<summary><b>💻 Quantum Programming Frameworks</b></summary>

- **Qiskit**: QuantumCircuit, AerSimulator, Primitives (Sampler, Estimator), transpiler, IBM Quantum Runtime
- **PennyLane**: QNode, devices, parameter-shift gradients, variational circuits, QML
- Framework interoperability and circuit conversion

</details>

<details>
<summary><b>🧠 Quantum Algorithms</b></summary>

- Quantum oracle construction and phase kickback
- Deutsch-Jozsa algorithm (exponential speedup, query complexity)
- Bernstein-Vazirani algorithm (exact 1-query solution)
- Simon's algorithm (exponential speedup, XOR periodicity, GF(2) linear algebra)
- Grover's search algorithm (quadratic speedup, amplitude amplification)

</details>

---

## 📖 References & Citations

1. Nielsen, M. A. & Chuang, I. L. (2010). *Quantum Computation and Quantum Information* (10th Anniversary Ed.). Cambridge University Press.
2. Preskill, J. (2018). Quantum Computing in the NISQ Era and Beyond. *Quantum*, 2, 79. https://doi.org/10.22331/q-2018-08-06-79
3. Wilde, M. M. (2017). *Quantum Information Theory* (2nd Ed.). Cambridge University Press.
4. Grover, L. K. (1996). A fast quantum mechanical algorithm for database search. *Proceedings of STOC*, 212–219. https://arxiv.org/abs/quant-ph/9605043
5. Simon, D. R. (1997). On the power of quantum computation. *SIAM Journal on Computing*, 26(5), 1474–1483.
6. Bernstein, E. & Vazirani, U. (1997). Quantum complexity theory. *SIAM Journal on Computing*, 26(5), 1411–1473.
7. Deutsch, D. & Jozsa, R. (1992). Rapid solution of problems by quantum computation. *Proceedings of the Royal Society A*, 439, 553–558.
8. Barenco, A. et al. (1995). Elementary gates for quantum computation. *Physical Review A*, 52(5), 3457. https://arxiv.org/abs/quant-ph/9503016
9. Bergholm, V. et al. (2022). PennyLane: Automatic differentiation of hybrid quantum-classical computations. https://arxiv.org/abs/1811.04968
10. IBM Qiskit Documentation. https://docs.quantum.ibm.com
11. PennyLane Documentation. https://docs.pennylane.ai
12. Cerezo, M. et al. (2021). Variational Quantum Algorithms. *Nature Reviews Physics*, 3, 625–644. https://arxiv.org/abs/2012.09265

---

## 🏷️ Topics & Tags

```
quantum-computing  qiskit  pennylane  quantum-algorithms  jupyter-notebook
python  quantum-gates  grover-algorithm  simons-algorithm  bernstein-vazirani
deutsch-jozsa  quantum-circuits  bloch-sphere  density-matrix  entanglement
quantum-noise  quantum-information  quantum-machine-learning  variational-quantum
amplitude-amplification  quantum-oracles  quantum-hackathon  nisq  quantum-education
trinexis  quantum-course  quantum-research  stern-gerlach  double-slit  phase-kickback
```

<!-- SEO KEYWORDS
quantum computing tutorial, qiskit tutorial, pennylane tutorial, quantum gates explained,
grover's algorithm python, simon's algorithm qiskit, bernstein vazirani algorithm,
deutsch jozsa algorithm, quantum circuits jupyter, bloch sphere visualisation,
density matrix python, quantum entanglement code, quantum noise models, kraus operators,
quantum information theory, variational quantum eigensolver, quantum machine learning,
quantum hackathon preparation, IBM quantum computing, quantum computing for students,
quantum computing research, NISQ algorithms, quantum computing roadmap, hands-on quantum,
Quantum Computing Education Series by Trinexis, quantum computing course 2026, quantum superposition code,
quantum measurement, bra-ket notation, quantum computing from scratch
-->

---

## 🌐 Trinexis

This series is brought to you by **[Trinexis](https://trinexis.org/)** - visit us at **https://trinexis.org/** for more quantum computing resources, programmes, and updates.

---

## 📄 License

This project is licensed under the **MIT License** - see [LICENSE](LICENSE) for details. You are free to use, modify, and distribute this material for educational purposes.

---

## 🙏 Acknowledgements

This notebook series was developed for the **Quantum Computing Education Series by Trinexis** held in **2026**, by **Trinexis Quantum**. We thank all participants and the open-source quantum computing community, especially the Qiskit and PennyLane teams.

---

<div align="center">

**[Trinexis Quantum](https://github.com/Trinexis-Quantum)**

</div>
