# 🌀 Hands-On 13 · The Bernstein–Vazirani Algorithm

[![Python](https://img.shields.io/badge/Python-3.8%2B-3776AB?logo=python&logoColor=white)](https://www.python.org/)
[![Qiskit](https://img.shields.io/badge/Qiskit-2.5.0-6929C4?logo=qiskit&logoColor=white)](https://qiskit.org/)
[![Qiskit Aer](https://img.shields.io/badge/Qiskit--Aer-0.17.2-6929C4)](https://qiskit.github.io/qiskit-aer/)
[![Jupyter](https://img.shields.io/badge/Jupyter-Notebook-F37626?logo=jupyter&logoColor=white)](https://jupyter.org/)
[![NumPy](https://img.shields.io/badge/NumPy-1.24%2B-013243?logo=numpy&logoColor=white)](https://numpy.org/)
[![License](https://img.shields.io/badge/License-MIT-22c55e)](LICENSE)

**Education Series · Quantum Computing · Trinexis**
*Hands-On Demo 13 of 15, Algorithms Module*

---

## Overview

Imagine you are given a sealed black box that accepts any $n$-bit input you care to send it and returns a single bit back. Inside, it computes the dot product (modulo 2) of your input with a secret $n$-bit string $s$ that is invisible to you. Your mission: find $s$. Classically, you would interrogate the box one bit at a time, send `100...0`, note the output, then `010...0`, and so on, requiring $n$ queries to pin down all $n$ bits. There is no shortcut: each query returns exactly one bit, and there are $n$ bits to learn.

The **Bernstein–Vazirani algorithm** recovers that entire secret string in a single oracle query, deterministically, no matter how large $n$ is. It does this not through magic but through a beautifully engineered **interference pattern**. Think of it like a precision-tuned acoustic room: when the room is shaped just right, every echo from the wrong answer cancels itself out, and only the correct answer comes back to you, loud and clear, with no ambiguity. The algorithm is a quantum interferometer in disguise. The two active ingredients are **phase kickback** (the oracle writes the secret into the *signs* of probability amplitudes rather than into any directly readable quantity) and **Hadamard interference** (a second wave of Hadamard gates makes every wrong-answer amplitude cancel to zero and the right-answer amplitude surge to one).

This notebook is the clearest demonstration of the course's central thesis, *indistinguishable alternatives interfere; probabilities are squared magnitudes of amplitudes*, in the entire algorithms module. You will build the algorithm twice: once from first principles using nothing but NumPy arrays, so every amplitude is visible, and once as a real quantum circuit in Qiskit running on the Aer simulator. You will watch the interference happen, test it at every scale from one to four qubits, stress-test it under simulated hardware noise, and solve four graded exercises that cement the ideas before you move on to Simon's algorithm.

---

## Learning Objectives

After completing this notebook you will be able to:

- State the **Bernstein–Vazirani problem** precisely: what the oracle computes, what the goal is, and why the classical lower bound is exactly $n$ queries.
- Explain, without resorting to "parallel universes" or vague "quantum parallelism," how **phase kickback** writes the secret $s$ into the *signs* of amplitudes.
- Show algebraically why the **final layer of Hadamard gates** makes every wrong-answer amplitude vanish through destructive interference, leaving only $|s\rangle$ with unit probability.
- Implement BV **from scratch in NumPy**, building the Hadamard tensor product, the phase-kickback oracle diagonal, and the three-step algorithm, and read off the hidden string from the output statevector.
- Construct a **Qiskit circuit** for BV with an explicit ancilla qubit, apply the CX-gate oracle, and handle the little-endian measurement convention correctly.
- Recognise how **depolarising noise** degrades the interference and shrinks the success probability, and explain intuitively *why* the interference is fragile.
- Use **majority voting** over many shots to recover reliability under moderate noise.
- Predict the measurement outcome for any hidden string *before* running the circuit.

---

## Background and Theory

### The Classical Problem

Suppose a friend conceals a bit-string $s = s_0 s_1 \dots s_{n-1}$, where each $s_i \in \{0, 1\}$. They hand you a function, an oracle, that evaluates:

$$f(x) = s \cdot x = (s_0 x_0 \oplus s_1 x_1 \oplus \cdots \oplus s_{n-1} x_{n-1}) \bmod 2$$

This is the bitwise dot product of your query $x$ with the secret $s$, taken modulo 2 (so the answer is always 0 or 1). To find all of $s$ classically you query with the unit strings $e_0 = 10\dots0$, $e_1 = 010\dots0$, and so on. Each query reveals exactly one bit: $f(e_i) = s_i$. After $n$ queries you have assembled all of $s$. This is provably optimal classically, no clever strategy does better, because each query returns exactly one bit of information and there are $n$ bits to determine.

### The Quantum Shortcut: One Query Suffices

The Bernstein–Vazirani algorithm recovers all $n$ bits of $s$ with a **single oracle call**, with certainty. This is a clean, unconditional query-complexity separation: $n$ classical queries versus $1$ quantum query.

The algorithm operates on $n + 1$ qubits: an $n$-qubit **input register** and a single **ancilla qubit**.

| Step | Operation | State of the input register |
|------|-----------|------------------------------|
| 0 | Initialise | $\lvert 0\rangle^{\otimes n}\lvert 1\rangle$ |
| 1 | $H$ on all qubits | $\frac{1}{\sqrt{2^n}}\sum_x \lvert x\rangle \otimes \lvert{-}\rangle$ |
| 2 | Oracle $U_f$ (kickback) | $\frac{1}{\sqrt{2^n}}\sum_x (-1)^{s \cdot x}\lvert x\rangle \otimes \lvert{-}\rangle$ |
| 3 | $H^{\otimes n}$ on inputs | $\lvert s\rangle \otimes \lvert{-}\rangle$ |
| 4 | Measure inputs | $s$ with probability 1 |

### Ingredient 1, Phase Kickback

Setting the ancilla to $\lvert 1 \rangle$ and applying $H$ turns it into $\lvert{-}\rangle = \frac{1}{\sqrt{2}}(\lvert 0\rangle, \lvert 1\rangle)$. When the oracle would flip the ancilla (because $f(x) = 1$), it instead leaves the ancilla unchanged and multiplies the input's amplitude by $-1$:

$$\lvert x \rangle \lvert{-}\rangle \;\xrightarrow{U_f}\; (-1)^{f(x)} \lvert x \rangle \lvert{-}\rangle$$

The ancilla is untouched; the phase is "kicked back" onto the input. After one oracle call the input register holds a uniform superposition with the secret encoded purely in the *signs* of the amplitudes, invisible to direct measurement, but fully accessible to the right interference operation.

### Ingredient 2, Hadamard Interference

The key algebraic fact is that the Hadamard transform of $\lvert s \rangle$ produces exactly the phase pattern that the oracle just created:

$$H^{\otimes n} \lvert s \rangle = \frac{1}{\sqrt{2^n}} \sum_x (-1)^{s \cdot x} \lvert x \rangle$$

Because $H^{\otimes n}$ is its own inverse ($H^{\otimes n} H^{\otimes n} = I$), applying it again undoes this transformation and lands the state on $\lvert s \rangle$ with certainty. The algebra that makes every wrong answer vanish is a discrete orthogonality relation:

$$H^{\otimes n}\!\left[\frac{1}{\sqrt{2^n}}\sum_{x}(-1)^{s\cdot x}\lvert x\rangle\right] = \frac{1}{2^n}\sum_{y}\underbrace{\left[\sum_{x}(-1)^{(s \oplus y)\cdot x}\right]}_{=\,2^n \text{ if } y=s,\;\;0 \text{ otherwise}}\lvert y\rangle = \lvert s\rangle$$

The bracketed sum is $2^n$ when $y = s$ (all $2^n$ terms are $+1$) and exactly zero for every other $y$ (positive and negative terms cancel pairwise). That cancellation, not any notion of parallel computation, is the entire mechanism of the algorithm.

### Analogy: The Acoustic Room

Think of the algorithm as a specially shaped concert hall. When you clap your hands (the oracle call), the sound bounces off every wall simultaneously. The hall is engineered so that echoes from every corner cancel each other through destructive interference, except for the one direction where they all arrive in phase and add constructively. A listener standing in that special spot hears a clear, amplified signal; everywhere else is silence. The hidden string $s$ is the geometry of the room; the final Hadamard layer is the engineering that routes the echoes. One clap tells you the complete geometry.

### Connection to the Course Thesis

This algorithm is the course thesis made computational: *indistinguishable alternatives interfere; probabilities are squared magnitudes of amplitudes.* The paths to wrong answers $y \neq s$ are indistinguishable-then-cancelled routes through the computation, exactly the destructive interference of the double-slit experiment from Demo 1, now harnessed as a computational tool.

### Where BV Sits in the Algorithm Family

- **Deutsch's problem** (one qubit), decide if $f(0) = f(1)$ with one query; the original kickback demonstration.
- **Deutsch–Jozsa** (Demo 12b), decide *constant vs balanced* with one query; same skeleton.
- **Bernstein–Vazirani** (this notebook), extract a full *linear structure* $s$ with one query.
- **Simon's algorithm** (Demo 14), find a *hidden period* using $O(n)$ queries and classical post-processing; the bridge to Shor.
- **Shor's algorithm**, find the period of modular exponentiation; the exponential speedup that drives quantum cryptography.

---

## Prerequisites

**Quantum mechanics concepts:**
- Qubit states, bra-ket notation, superposition (Demo 3)
- The Bloch sphere and single-qubit gates (Demo 4)
- Tensor products, multi-qubit states, entanglement (Demo 5)
- Hadamard gate as a basis-change operation (Demo 7–8)
- Decoherence and the effect of noise on quantum states (Demo 6)

**Algorithms background:**
- Deutsch–Jozsa algorithm and the concept of a quantum oracle (Demo 12b)
- Basic query complexity: what it means to "query" an oracle

**Programming and tools:**
- Python 3.8+ with NumPy (linear algebra operations, `np.kron`, matrix multiplication)
- Jupyter Notebook or Google Colab
- Qiskit 2.5.0 and Qiskit-Aer 0.17.2 (installed automatically in the setup cell)
- Familiarity with `QuantumCircuit`, `transpile`, and `AerSimulator` from Demo 9

**Mathematics:**
- Complex numbers and their modulus
- Matrix–vector multiplication and the Kronecker product
- Modular arithmetic (specifically arithmetic modulo 2)
- Basic probability: probability as squared magnitude of amplitude

---

## Notebook Walkthrough

### Setup (Cell 5), Installing the Environment

The notebook opens with a guard-gated pip install: it imports Qiskit and Qiskit-Aer and, only if they are absent, fetches the exact pinned versions (`qiskit==2.5.0`, `qiskit-aer==0.17.2`). This reproducibility discipline, pinning versions, is important because quantum SDK APIs change rapidly. The cell also seeds NumPy for reproducibility and sets a display resolution for crisp figures.

---

### Part A, Build It from Scratch in NumPy

**Why NumPy first?** When you call a Qiskit function, you hand control to a library. The arithmetic is invisible. Building the statevector simulation by hand forces you to see every amplitude, every sign flip, every matrix multiplication. If BV is to become genuinely understood rather than trusted on authority, this transparency is indispensable.

#### Section A.1, The Reusable Toolkit (Cells 7)

Seven utility functions are defined, each doing one small thing:

- `int_to_bits(x, n)`, converts an integer to an MSB-first bit array. The MSB-first convention (bit 0 is the most significant) is maintained uniformly across the series; violating it is the single most common source of confusion in multi-qubit programming.
- `bits_to_str` / `str_to_bits`, conversions between array and string representations of bit-strings.
- `dot_mod2(s_bits, x_bits)`, computes $s \cdot x \bmod 2$, the oracle function itself.
- `hadamard_n(n)`, builds $H^{\otimes n}$ as a dense matrix via repeated `np.kron`. For $n = 3$ this is an $8 \times 8$ matrix; for $n = 10$ it would be $1024 \times 1024$. The function works for any $n$ but is intended for small $n$ in a teaching context.
- `bv_oracle_phase(s_bits)`, produces the diagonal of the phase-kickback oracle: a length-$2^n$ vector where entry $x$ is $(-1)^{s \cdot x}$. Because the ancilla is in $\lvert{-}\rangle$ throughout, the full oracle matrix reduces to this diagonal action on the input register. Multiplying a state vector element-wise by this diagonal is exact and efficient.

This toolkit is carried forward without modification into every subsequent notebook in the series. Learning it once here pays dividends through Simon's and Grover's.

#### Section A.2, The Core BV Runner and the 1-Qubit Case (Cells 8–12)

`run_bv_numpy(s_bits)` executes the three-step algorithm:

1. `psi0 = [1, 0, ..., 0]`, the all-zeros computational basis state.
2. `psi1 = Hn @ psi0`, uniform superposition. Every amplitude is $1/\sqrt{2^n}$.
3. `psi2 = oracle * psi1`, the oracle flips some signs. The state is still uniform in *magnitude*, all amplitudes have the same size, but some are now negative. The secret $s$ is encoded entirely in this sign pattern. Crucially, you cannot read $s$ from `psi2` by measuring: you would get a random outcome with equal probability.
4. `psi3 = Hn @ psi2`, the second Hadamard layer exploits the interference to convert that sign pattern back into a single basis state. The peak of `|psi3|^2` identifies $s$.

For $n = 1$: there are only two secrets and two basis states. With `s = "1"`, after the oracle the amplitudes are `[+1/√2, -1/√2]`. After the final Hadamard, these recombine to `[0, 1]`, meaning $\lvert 1\rangle$ with certainty. The interference visualisation in Cell 12 makes this concrete: the left bar chart shows the spread-and-signed post-oracle amplitudes; the right chart shows the collapsed, single-bar post-interference result.

#### Section A.3, The 2-Qubit Case (Cells 13–14)

With $n = 2$ there are four possible secrets and $2^2 = 4$ basis states. The notebook runs all four and confirms each is recovered with `P(correct) = 1.000`. The interference plot for `s = "11"` shows a four-element sign pattern before the final Hadamard and a perfect spike at `|11⟩` after it. Notice that the pattern is different for each secret, this is the oracle encoding different information, yet the second Hadamard always extracts it perfectly.

#### Section A.4, The 3-Qubit Case (Cells 15–16)

Eight possible secrets, all swept in a single loop. Every single one is recovered with zero error. The notebook includes an explicit pass/fail check (`all_ok`) rather than relying on visual inspection, a habit of rigorous testing that students should adopt. The interference plot for `s = "101"` is particularly instructive: eight amplitudes before the final Hadamard form a checkerboard of $+$ and $-$ signs that encode the string `101`; after the Hadamard, six of the eight amplitudes cancel to zero and the amplitude at index 5 (binary `101`) reaches exactly 1.

#### Section A.5, Classical vs Quantum Query Count (Cell 18)

This section makes the advantage concrete by running the honest classical algorithm, probe with $e_i$ one at a time, collect one bit per query, and tabulating the query counts side by side. For $n = 1$: classical 1, quantum 1 (no advantage yet). For $n = 2$: classical 2, quantum 1. For $n = 3$: classical 3, quantum 1. The gap is not yet exponential (that comes with Shor), but it is clean, unconditional, and deterministic, which is why BV is the canonical introductory example of a quantum query advantage.

---

### Part B, The Same Algorithm in Qiskit

**Why build it again?** The NumPy version is a model, a mathematical object. The Qiskit version is a circuit, a sequence of physical operations on qubits. Seeing that they agree is not just reassuring; it demonstrates that the abstract mathematics and the actual gate model are the same thing.

#### Section B.1, The Circuit Builder (Cell 20)

`bv_circuit(s_bits)` builds the circuit in five logical moves:

1. `qc.x(n)`, flip the ancilla (qubit $n$) to $\lvert 1\rangle$.
2. `qc.h(range(n+1))`, Hadamard on all $n+1$ qubits. The input register becomes a uniform superposition; the ancilla becomes $\lvert{-}\rangle$.
3. Oracle loop, for each bit position $i$ with $s_i = 1$, add a CX gate from input qubit $i$ to the ancilla. This is exactly the controlled-NOT that implements $f(x) = s \cdot x$; when the ancilla is $\lvert{-}\rangle$, the CX's phase kickback writes $(-1)^{x_i}$ onto qubit $i$.
4. `qc.h(range(n))`, Hadamard on the input register only. This is the interference step.
5. `qc.measure(range(n), range(n))`, measure the input register.

**The endianness note is critical.** Qiskit's measurement string is little-endian: the leftmost character of a counts key corresponds to the highest-index qubit. The hidden string follows the course's MSB-first convention: character $k$ corresponds to qubit $k$. Reversing the counts key (`top[::-1]`) converts from Qiskit's convention to the course convention. Neglecting this reversal causes every output to look wrong on multi-qubit cases. The notebook flags this prominently because it is a perennial source of bugs.

#### Section B.2, Sweeping All Sizes (Cell 22)

The Aer simulator with 1024 shots confirms every secret for $n = 1, 2, 3$. All shots land on the correct string, 100% accuracy, because the ideal simulator has no noise. This validates the circuit construction and the endianness handling simultaneously.

#### Section B.3, The Measurement Histogram (Cell 24)

`plot_counts` draws a bar chart of all $2^n$ possible outcomes. On an ideal simulator the chart is maximally boring: a single red bar at $s$, everything else at zero. This is the point. A quantum algorithm that has worked perfectly is sometimes visually dull, and that is the best possible sign.

#### Section B.4, Cross-Checking NumPy Against Qiskit (Cell 26)

Both independent implementations are run on every secret for $n = 1, 2, 3$ and their outputs compared. The fact that they agree on all $14$ test cases (including the all-zeros secret, which is a common edge case) provides strong evidence that neither implementation contains a hidden error. This pattern of maintaining two independent implementations and cross-checking them is standard practice in algorithm development.

---

### Part C, What Noise Does

#### The Physical Picture (Cell 28)

Real quantum hardware is not perfect. Gates apply slightly wrong operations; qubits interact with their environment. These effects are modelled here by **depolarising noise**, after each gate, the qubit is replaced by a random Pauli operator with probability $p$. This scrambles the precise phase relationships that the interference depends on.

At $p = 0.05$ (5% error per gate) the 3-qubit BV circuit's success probability drops from 1.000 to roughly 0.810. The histogram is no longer a single spike: wrong-answer strings now carry a fraction of the shots. The correct string is still the *tallest* bar, the algorithm degrades gracefully, but the determinism is gone.

This is not a failure of the algorithm. It is a demonstration of why **quantum error correction** is the grand engineering challenge of the field. The phase relationships that constitute the algorithm's advantage are the same phase relationships that decoherence destroys.

#### Noise Scaling (Cell 29)

A sweep across per-gate error rates from 0% to 20% shows the success probability curve. At 0%: perfect. At about 5%: still clearly above chance (which for $n = 3$ is $1/8 = 12.5\%$). At 20%: indistinguishable from random guessing. The dashed reference line at $1/8$ marks the threshold below which the algorithm has lost its advantage entirely.

---

### Part D, Exercises

Four exercises, graded from one to four stars, each with a deliberately wrong placeholder (returning `-999` or `"???"`) so that an incomplete attempt is immediately visible rather than silently wrong:

- **Exercise 1 (★):** Predict the outcome of `s = "110"` before running. Tests conceptual understanding.
- **Exercise 2 (★★):** Reimplement `bv_oracle_phase` from scratch using `int_to_bits` and `dot_mod2`, without calling the existing function. Builds understanding of what the phase oracle actually computes.
- **Exercise 3 (★★★):** Given only the phase vector of a mystery oracle (not the hidden string itself), recover $s$ by applying the BV procedure. This is the closest this notebook comes to "real" oracle problem-solving: you are given the oracle's effect, not its internals.
- **Exercise 4 (★★★★):** Under 8% depolarising noise, use majority voting over many shots to recover reliability. Implement `majority_recover` by finding the most frequent outcome in the counts dictionary. This is the seed of the idea behind repetition codes and, conceptually, error correction.

Each exercise has a collapsed solution block. The recommendation is to try for at least 20 minutes before opening it.

---

### Part E, Play with It

The sandbox cell is the most important part of the notebook for building intuition. Change `HIDDEN` to any string, predict the outcome (you should always be able to for BV), then run. The recommended experiments are:

- **Flip one bit of `HIDDEN`:** the interference picture changes completely, but the algorithm still works. Why?
- **Try `"000"`:** the oracle does nothing ($f(x) = 0$ for all $x$), but the algorithm still correctly identifies the all-zeros secret. Trace through the algebra.
- **Try a long string like `"10110100"`:** the circuit grows in proportion to the number of 1-bits in $s$ (one CX gate per 1-bit), but the query count remains exactly 1.
- **Explore noise:** what per-gate error rate makes the histogram "too noisy to trust"? Is there a sharp threshold or a gradual degradation?

---

## Key Takeaways

- **The BV problem** is to find a hidden $n$-bit string $s$ given only an oracle for $f(x) = s \cdot x \bmod 2$. Classical algorithms need $n$ queries; BV needs exactly 1.
- **Phase kickback** is the mechanism by which the oracle encodes $s$ into the signs (phases) of probability amplitudes. The ancilla in $\lvert{-}\rangle$ acts as the phase "receiver." No measurement during this step reveals $s$; the information is in the phase structure, not the magnitudes.
- **The final Hadamard layer is an interferometer.** It is not a second "query", it is a wave-optics recombiner that constructively reinforces the correct answer and destructively cancels every wrong one. The cancellation is exact in an ideal system.
- **Quantum advantage here is not about parallelism.** Preparing a superposition and measuring gives random noise. The advantage comes entirely from the engineered phase structure that the oracle creates and the final Hadamard resolves.
- **Two independent implementations (NumPy and Qiskit) agree on all test cases**, providing high confidence in both the algorithm and the code. This cross-checking habit is essential for any quantum software development.
- **Noise breaks interference.** Depolarising errors scramble the precise phase relationships. Success probability degrades smoothly with error rate; the correct string remains the mode under moderate noise and can be extracted by majority voting.
- **BV is the middle of a family.** It extends Deutsch–Jozsa (one global property) and is extended by Simon's algorithm (hidden period, entanglement). Every member uses the same kickback-plus-interference skeleton. Mastering BV makes the rest of the family immediately accessible.

---

## Real-World Applications

Understanding Bernstein-Vazirani Algorithm is not just theoretical. Here is how it connects to active real-world problems and solutions:

- **Hidden Pattern Recovery**: BV's guaranteed single-query recovery of a hidden string has direct analogy in machine learning feature selection, where hidden structure in data must be recovered efficiently.
- **Quantum Linear Algebra**: The GF(2) inner product structure in BV is a building block for quantum linear systems algorithms used in data fitting, fluid simulations, and financial modelling.
- **Cryptographic Key Recovery**: BV demonstrates the principle behind quantum attacks on linear cryptographic systems, directly relevant to post-quantum cryptography standardisation efforts (NIST PQC).
- **Quantum Communication Complexity**: BV is a key example in quantum communication complexity theory, with implications for distributed computing and network protocol design.
- **Quantum Error Syndrome Decoding**: The syndrome measurement in quantum error-correcting codes is structurally identical to the BV oracle evaluation, making this notebook foundational for quantum fault-tolerance research.
- **Quantum Error Correction (QEC)**: The syndrome measurement circuit in stabiliser-based QEC is structurally identical to BV oracle evaluation; understanding BV is a direct conceptual stepping stone to implementing parity check measurements in surface codes.
- **AI and ML**: Hidden linear structure recovery in BV directly maps to quantum-enhanced feature detection in ML; quantum linear algebra subroutines (HHL algorithm) use similar inner product evaluation to BV for data fitting, recommendation systems, and anomaly detection.
- **Healthcare**: Quantum linear algebra applications enabled by BV-style circuits include quantum-accelerated analysis of electronic health records (EHR), multi-omics data integration, and quantum-enhanced imaging reconstruction algorithms for MRI and CT.

---

## Further Reading and Citations

1. **Bernstein, E., & Vazirani, U. (1997).** Quantum Complexity Theory. *SIAM Journal on Computing*, 26(5), 1411–1473. [https://doi.org/10.1137/S0097539796300921](https://doi.org/10.1137/S0097539796300921)
   *The original paper proving the BV result and establishing the quantum query complexity framework. Section 3 gives the precise formulation used in this notebook.*

2. **Nielsen, M. A., & Chuang, I. L. (2010).** *Quantum Computation and Quantum Information* (10th anniversary ed.). Cambridge University Press. [https://doi.org/10.1017/CBO9780511976667](https://doi.org/10.1017/CBO9780511976667)
   *The standard reference. Chapter 1.4 covers Deutsch's and BV algorithms; Chapter 6 gives the full query complexity treatment. Essential reading before Simon's and Grover's.*

3. **Mermin, N. D. (2007).** *Quantum Computer Science: An Introduction*. Cambridge University Press. [https://doi.org/10.1017/CBO9780511813870](https://doi.org/10.1017/CBO9780511813870)
   *Excellent for physicists and computer scientists alike. Chapter 2 gives an unusually clear treatment of the BV and Deutsch–Jozsa algorithms with careful attention to the oracle model.*

4. **Montanaro, A. (2016).** Quantum algorithms: an overview. *npj Quantum Information*, 2, 15023. [https://doi.org/10.1038/npjqi.2015.23](https://doi.org/10.1038/npjqi.2015.23)
   *A concise, authoritative survey of where quantum algorithms provide proven advantage. BV is discussed in Section 2 as the canonical linear-structure example. Freely available via arXiv:1511.04206.*

5. **Cleve, R., Ekert, A., Macchiavello, C., & Mosca, M. (1998).** Quantum algorithms revisited. *Proceedings of the Royal Society A*, 454(1969), 339–354. [https://doi.org/10.1098/rspa.1998.0164](https://doi.org/10.1098/rspa.1998.0164)
   *Reformulates Deutsch–Jozsa and BV in the now-standard phase-kickback framework. Essential for understanding why the ancilla is prepared in $\lvert{-}\rangle$ rather than some other state.*

6. **Childs, A. M. (2021).** Lecture Notes on Quantum Algorithms. University of Maryland. [https://www.cs.umd.edu/~amchilds/qa/](https://www.cs.umd.edu/~amchilds/qa/)
   *Freely available course notes that cover BV (Lecture 2), Simon's, and Shor's with precise complexity statements and complete proofs. Ideal supplement before tackling Demo 14.*

7. **IBM Quantum Learning, Bernstein–Vazirani Algorithm.** [https://learning.quantum.ibm.com/tutorial/bernstein-vazirani-algorithm](https://learning.quantum.ibm.com/tutorial/bernstein-vazirani-algorithm)
   *Interactive Qiskit implementation with runtime execution on real IBM quantum hardware. Compare the noise profiles you see in Part C of this notebook against a live device.*

---

<!--
SEO TAGS, DO NOT REMOVE
Bernstein-Vazirani algorithm, quantum algorithm, quantum oracle, phase kickback, Hadamard gate, quantum interference, quantum query complexity, quantum speedup, Qiskit tutorial, quantum computing education, quantum superposition, destructive interference, constructive interference, qubit, quantum gate, quantum circuit, Trinexis, quantum computing education series by Trinexis 2026, Qiskit Aer, depolarising noise, quantum error, majority voting, hidden string problem, dot product oracle, quantum parallelism misconception, quantum information, quantum computing Python, computational basis, Walsh-Hadamard transform, oracle model, quantum advantage, classical vs quantum
-->

---

## Related Notebooks in This Series

| # | Notebook | Topic |
|---|----------|-------|
| 1–2 | [Demo1-2\_Double\_Slit\_and\_Stern\_Gerlach\.ipynb](Demo1-2_Double_Slit_and_Stern_Gerlach.ipynb) | Double-slit interference and Stern–Gerlach; the physical basis of amplitude interference |
| 3 | [Demo3\_QMPostulates\_BraKet\_Bloch.ipynb](Demo3_QMPostulates_BraKet_Bloch.ipynb) | Quantum mechanics postulates, bra-ket notation, Bloch sphere |
| 4 | [Demo4\_BlochSphere\_DensityMatrix.ipynb](Demo4_BlochSphere_DensityMatrix.ipynb) | Density matrices and the geometry of mixed states |
| 5 | [Demo5\_Purity\_Coherence\_Entanglement.ipynb](Demo5_Purity_Coherence_Entanglement.ipynb) | Entanglement measures, purity, and coherence |
| 6 | [Demo6\_Noise\_and\_Information\_Measures.ipynb](Demo6_Noise_and_Information_Measures.ipynb) | Quantum noise channels and information theory; foundation for Part C of this notebook |
| 7 | [Demo7\_Quantum\_Gates\_Demo.ipynb](Demo7_Quantum_Gates_Demo.ipynb) | Single- and multi-qubit gate zoo, including the Hadamard gate |
| 8 | [Demo8\_QuantumCircuits\_EntanglingGates\_WHT.ipynb](Demo8_QuantumCircuits_EntanglingGates_WHT.ipynb) | Quantum circuits, CX gate, and the Walsh–Hadamard transform underpinning BV |
| 9 | [Demo9\_Qiskit\_Introduction.ipynb](Demo9_Qiskit_Introduction.ipynb) | First Qiskit circuits, transpilation, and measurement |
| 10 | [Demo10\_PennyLane\_Introduction\_Hands\_On.ipynb](Demo10_PennyLane_Introduction_Hands_On.ipynb) | PennyLane framework introduction |
| 11 | [Demo11\_Qiskit\_PennyLane\_Synthesis\_and\_Comparison.ipynb](Demo11_Qiskit_PennyLane_Synthesis_and_Comparison.ipynb) | Cross-framework comparison: Qiskit vs PennyLane |
| **12** | [**Demo12b\_Qiskit\_Oracles\_Primitives\_DJ.ipynb**](Demo12b_Qiskit_Oracles_Primitives_DJ.ipynb) | **Quantum oracles and Deutsch–Jozsa, direct predecessor to this notebook** |
| **13** | **Demo13b\_Bernstein\_Vazirani\_Qiskit.ipynb** | **This notebook** |
| 14 | [Demo14\_Simons\_Algorithm\_Qiskit.ipynb](Demo14_Simons_Algorithm_Qiskit.ipynb) | Simon's algorithm: hidden-period finding, the conceptual bridge to Shor |
| 15 | [Demo15\_Grover\_Qiskit\.ipynb](Demo15_Grover_Qiskit.ipynb) | Grover's search algorithm: amplitude amplification and the quadratic speedup |

---

*Quantum Computing Education Series by Trinexis 2026*
*Quantum Computing Education Series by Trinexis 2026*
