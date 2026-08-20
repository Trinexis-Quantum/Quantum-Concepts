# Hands-On 14 · Simon's Algorithm with Qiskit 🔮

![Python](https://img.shields.io/badge/Python-3.9%2B-blue?logo=python&logoColor=white)
![Qiskit](https://img.shields.io/badge/Qiskit-2.5.0-6929C4?logo=qiskit&logoColor=white)
![Jupyter](https://img.shields.io/badge/Jupyter-Notebook-F37626?logo=jupyter&logoColor=white)
![License](https://img.shields.io/badge/License-MIT-green?logo=opensourceinitiative&logoColor=white)

> **Education Series · Quantum Computing · Trinexis**

---

## Overview

Imagine you have a mysterious black box - a machine that takes an $n$-bit string as input and spits out another $n$-bit string as output. Someone has secretly programmed it so that exactly two different inputs always produce the same output: if input $x$ gives output $w$, then input $x \oplus s$ (where $s$ is a hidden "secret" string and $\oplus$ is bitwise XOR) also gives $w$. Your job is to figure out what $s$ is, using as few queries to the box as possible. This is **Simon's Problem** - and it sits at the very heart of quantum computing's advantage over classical machines.

A classical computer trying to crack this puzzle must play a guessing game. Outputs look random, so you have no choice but to query the box repeatedly and hope that two of your queries happen to land on a colliding pair. By a birthday-paradox argument, you are likely to need around $\Omega(2^{n/2})$ queries - a number that grows *exponentially* with $n$. For even a modest $n = 50$, that means more queries than there are atoms in your body. **Simon's quantum algorithm, by contrast, solves the same problem with only $O(n)$ queries** - a number that grows merely linearly. That exponential gap was, when Simon published it in 1994, the first *provable* superpolynomial separation between quantum and classical query complexity. Think of it like this: the classical approach is searching for a matching pair of socks in a dark room by picking up one at a time; the quantum approach turns on a diffuse light that makes matching pairs literally glow, so you notice them all at once.

The magic ingredient is **quantum interference**. When you query the oracle in superposition, both $x$ and $x \oplus s$ are in flight simultaneously and both write the *same* output value. Because their contributions to any downstream measurement are now *indistinguishable*, their quantum amplitudes add together. A second round of Hadamard gates then steers that combined amplitude: for every candidate measurement outcome $z$ where $z \cdot s \equiv 1 \pmod{2}$, the two contributions arrive with *opposite signs* and cancel to zero (destructive interference). For every $z$ where $z \cdot s \equiv 0 \pmod{2}$, they add and survive (constructive interference). Each run of the circuit therefore hands you a random linear equation about $s$ - for free. Collect $n-1$ independent equations, solve a tiny system of linear algebra over GF(2), and you have $s$. This notebook builds the whole story from first principles, in both bare NumPy and in Qiskit, so the interference is never hidden behind a framework.

---

## Learning Objectives

By the end of this notebook you will be able to:

- State Simon's Problem precisely, including the two-to-one / one-to-one promise, and explain why it is exponentially hard for a classical computer.
- Trace through the five-step quantum circuit (superposition → oracle → Hadamard → measure → solve) and explain what each step accomplishes *physically*.
- Derive the interference formula showing which measurement outcomes are destroyed and which survive, and connect it to the course thesis: *indistinguishable alternatives interfere; probabilities are squared magnitudes of amplitudes*.
- Implement a Simon oracle from scratch as a NumPy lookup table and as a Qiskit quantum gate, and verify the two-to-one promise programmatically.
- Perform Gaussian elimination (reduced row echelon form) over GF(2) to extract the null-space vector that encodes the secret $s$.
- Run Simon's circuit on both the ideal Aer simulator and a depolarizing-noise model, interpret the resulting count histograms, and apply a majority-vote robustness strategy.
- Explain Simon's algorithm's relationship to Shor's algorithm and the hidden-subgroup problem framework.

---

## Background & Theory

### Simon's Problem

A function $f : \{0,1\}^n \to \{0,1\}^n$ is promised to satisfy:

$$f(x) = f(y) \quad \Longleftrightarrow \quad y = x \oplus s$$

for a fixed, unknown **secret string** $s \in \{0,1\}^n$. Here $\oplus$ denotes bitwise XOR (addition mod 2 componentwise).

- When $s \neq 0^n$, the function is **two-to-one**: each output value is hit by exactly the two inputs $\{x,\; x \oplus s\}$, which form a *coset* of the subgroup $\{0^n, s\}$ inside $(\mathbb{Z}_2)^n$.
- When $s = 0^n$, the function is **one-to-one** (a bijection); there is no hidden structure.

**Goal:** Determine $s$ using as few oracle queries as possible.

### Classical Complexity

A classical algorithm learns nothing from a single query. To find a *collision* (two inputs $x \neq y$ with $f(x) = f(y)$), which immediately reveals $s = x \oplus y$, it must sample enough inputs that two of them land in the same coset. By the birthday paradox, this requires $\Omega(2^{n/2})$ queries in the worst case - exponential in $n$.

### The Quantum Circuit

The Simon circuit uses two $n$-qubit registers - an *input* register and an *output* register - and proceeds in five steps:

**Step 1 - Create a uniform superposition** by applying $H^{\otimes n}$ to the input register:

$$|\psi_1\rangle = \frac{1}{\sqrt{2^n}} \sum_{x \in \{0,1\}^n} |x\rangle |0\rangle$$

**Step 2 - Query the oracle** $U_f : |x\rangle|y\rangle \mapsto |x\rangle|y \oplus f(x)\rangle$:

$$|\psi_2\rangle = \frac{1}{\sqrt{2^n}} \sum_{x} |x\rangle |f(x)\rangle$$

The input and output registers are now entangled. Crucially, inputs $x_0$ and $x_0 \oplus s$ carry the *same* output label - they have become indistinguishable.

**Step 3 - Apply Hadamards again** to the input register. Using $H^{\otimes n}|x\rangle = \frac{1}{\sqrt{2^n}} \sum_z (-1)^{x \cdot z}|z\rangle$:

$$|\psi_3\rangle = \frac{1}{2^n} \sum_x \sum_z (-1)^{x \cdot z} |z\rangle |f(x)\rangle$$

**Step 4 - Compute the amplitude** of a specific outcome $|z\rangle|w\rangle$. For a two-to-one $f$ with image value $w$, the contributing inputs are exactly the pair $\{x_0,\; x_0 \oplus s\}$:

$$\text{amplitude of }|z\rangle|w\rangle = \frac{(-1)^{x_0 \cdot z}}{2^n}\Big[1 + (-1)^{s \cdot z}\Big]$$

The bracket is the entire story:

$$1 + (-1)^{s \cdot z} = \begin{cases} 2 & \text{if } s \cdot z \equiv 0 \pmod{2} \quad \textbf{(constructive interference)} \\ 0 & \text{if } s \cdot z \equiv 1 \pmod{2} \quad \textbf{(destructive interference)} \end{cases}$$

**Step 5 - Measure** the input register. Every outcome $z$ you can ever observe satisfies:

$$\boxed{z \cdot s \equiv 0 \pmod{2}}$$

and the surviving outcomes are *uniformly distributed* over the $(n-1)$-dimensional subspace perpendicular to $s$ in $(\mathbb{Z}_2)^n$.

### Classical Post-Processing: Solving Over GF(2)

Each circuit run produces a random $z$ from the orthogonal complement of $s$. Stack $n-1$ linearly independent measurements as rows of a matrix $Z$ and solve the homogeneous system:

$$Z \,\vec{s} = \vec{0} \pmod{2}$$

The null space of $Z$ over GF(2) is exactly $\{0^n, s\}$, so the unique non-zero solution is $s$. If instead the rows span the full $n$-dimensional space (rank = $n$), the null space is trivial, confirming $s = 0^n$ (one-to-one function). Gaussian elimination over GF(2) accomplishes this in $O(n^3)$ classical bit operations - negligible cost.

### Quantum Speedup Summary

| | Oracle Queries | Post-processing |
|---|---|---|
| **Classical (best)** | $\Omega(2^{n/2})$ | - |
| **Simon (quantum)** | $O(n)$ | $O(n^3)$ GF(2) solve |

This is an **exponential** separation in query complexity - the first ever proven for any computational problem. It directly inspired Shor's algorithm: replacing the group $(\mathbb{Z}_2)^n$ with $\mathbb{Z}_N$ and the second Hadamard with the Quantum Fourier Transform turns Simon's "find the XOR-period" into Shor's "find the multiplicative period," which enables integer factoring in polynomial time.

---

## Prerequisites

- **Python** basics: functions, NumPy arrays, list comprehensions.
- **Linear algebra**: matrix–vector multiplication, systems of equations, basis and null space (high-school level is enough; GF(2) arithmetic is introduced from scratch).
- **Quantum computing fundamentals** (covered in earlier hands-on notebooks):
  - Qubits, Dirac notation, superposition, measurement and the Born rule.
  - Single-qubit gates (X, H) and two-qubit gates (CNOT / CX).
  - Quantum circuits and the Aer simulator.
  - Tensor products and multi-qubit state vectors.
- **Optional but helpful**: familiarity with the concept of query complexity / oracle problems (Hands-On 12–13).

---

## Notebook Walkthrough

### Section 0 · Setup (cells 1–3)

The notebook opens with a top-level markdown cell that summarises the entire algorithm in one equation and one paragraph - a deliberate design choice to give readers an anchor before any code. The setup section then installs Qiskit 2.5.0 and qiskit-aer 0.17.2 *only if they are missing*, using a `try/except ImportError` guard so the cell is fast on re-runs and works equally well in Google Colab and local environments. A second cell imports NumPy and Matplotlib, fixes a global random seed (`np.random.seed(42)`) for reproducibility across the whole course, and sets tasteful Matplotlib defaults (figure size, grid transparency, font size).

**Why it matters:** Setting the seed ensures every student's runs produce identical outputs, which is important when students compare results or follow along with expected values in lecture.

---

### Section 1 · The Simon Problem (cell 4)

This markdown cell states the problem in full mathematical rigour - the XOR-period promise, both the two-to-one and one-to-one flavours, a concrete 3-qubit worked example ($s = 110_2$), and the birthday-paradox argument for the classical lower bound. It also provides historical context: Simon's 1994 result was the *first provable exponential separation* between quantum and classical query complexity, and it directly inspired Shor's 1994 factoring algorithm.

**Why it matters:** Anchoring the algorithm in a concrete problem and a clear classical baseline makes the quantum speedup feel meaningful rather than abstract. Students understand *why* we care before seeing a single gate.

---

### Section 2 · The Quantum Algorithm (cell 5)

A detailed five-step derivation of the circuit, written out in full Dirac notation with every intermediate state. The key interference equation:

$$1 + (-1)^{s \cdot z} \in \{0, 2\}$$

is boxed and called "the whole story," directing the student's attention to the single algebraic fact that explains the entire speedup. The section closes by connecting this to the course thesis (*indistinguishable alternatives interfere*) and describing the post-processing strategy.

**Why it matters:** Many quantum-computing texts gloss over this derivation. Writing it out step by step makes interference a *calculated* phenomenon rather than a mysterious one. Students who work through this section can reproduce the argument from memory.

---

### Section 3 · The Reusable Toolkit (cells 6–8)

Four small NumPy helper functions are introduced and immediately tested with `assert` statements:

- `int_to_bits(v, n)` / `bits_to_int(bits)`: convert between integers and LSB-first bit arrays (qubit $j$ carries weight $2^j$).
- `parity(x)`: XOR of all bits of an integer.
- `dot_mod2(a, b)`: bitwise inner product mod 2 - the core operation that decides whether a string $z$ survives or is cancelled.

The explicit LSB-first convention and its motivation (qubit $j$ in the array corresponds to qubit $j$ in the circuit) are explained in the surrounding markdown. This avoids the subtle endianness bugs that plague quantum-computing notebooks.

Then `make_simon_function(s, n)` builds a lookup table oracle by walking through inputs, assigning a fresh shuffled output label to each unvisited coset $\{x, x \oplus s\}$. The `verify_function` helper asserts the promise holds and checks the one-to-one / two-to-one condition, making it impossible to accidentally pass a broken oracle to the algorithm.

A concrete peek cell prints the full input/output table for $n=3$, $s=110_2$, showing each input, its output, and its colliding partner - exactly what an opaque oracle would hide from a classical algorithm.

**Why it matters:** Building oracles from scratch, rather than importing them from a library, forces the distinction between the *problem description* (the oracle's job) and the *algorithm's view* (it can only call `f`, not inspect its internals). The shuffle ensures the output looks random, mimicking a truly opaque black box.

---

### Section 4 · Statevector Simulation (cells 9–11)

Three functions implement the full Simon circuit as explicit matrix–vector algebra, with no Qiskit involved:

- `hadamard_n(n)`: builds $H^{\otimes n}$ via repeated `np.kron` products.
- `oracle_matrix(f, n)`: builds the $2^{2n} \times 2^{2n}$ permutation matrix for $U_f$ using the basis-indexing scheme `(x << n) | y`.
- `simon_marginal(f, n)`: runs $H^{\otimes n}_\text{in} \to U_f \to H^{\otimes n}_\text{in}$, applies the Born rule, and marginalises the output register to produce the measurement probability distribution over input-register outcomes $z$.

After computing the distribution for $n=3$, $s=110_2$, the notebook prints each $z$ with its probability and the value of $z \cdot s \pmod{2}$, then uses a hard `assert` to confirm that *every* string with $z \cdot s = 1$ has exactly zero probability. A bar chart colour-codes blue (survived) vs. red (would appear if interference were absent) to make the pattern visually obvious.

**Why it matters:** This section is the pedagogical centrepiece. By computing the amplitude formula numerically *and* algebraically, students see interference as a concrete mathematical fact. The hard assertion doubles as a unit test that would immediately catch any bug in the oracle or Hadamard construction.

---

### Section 5 · Classical Post-Processing Over GF(2) (cells 12–14)

Four functions implement the linear-algebra post-processing:

- `gf2_rref(rows, n)`: reduced row echelon form over GF(2) via Gaussian elimination, returning pivot columns.
- `gf2_nullspace(rows, n)`: extracts the null-space basis by back-substituting free variables.
- `solve_simon(zs, n)`: wraps the above to return the secret $s$ (or $0^n$ if the equations reach full rank).
- `run_simon(f, n, ...)`: the complete Simon driver - samples circuit runs by drawing from the marginal distribution, accumulates linearly independent equations, stops when rank reaches $n-1$ (or $n$ for the one-to-one case), and returns the recovered secret and all collected measurements.

A `demo_simon` wrapper ties everything together: build oracle, verify, marginalise, run, recover, check, and plot - all in one call with colour-coded output (blue: survived, red: cancelled).

**Why it matters:** Showing that the quantum algorithm's output requires a *classical* linear-algebra solver to extract $s$ is important. It prevents the misconception that quantum computers magically "read out" answers - the quantum step provides equations, the classical step solves them.

---

### Section 6 · Case n = 1 - The Degenerate Warm-Up (cells 15–16)

For $n=1$, the secret is a single bit. The notebook works through both sub-cases:

- **$s=1$ (two-to-one):** the oracle maps both 0 and 1 to the same value. The circuit *always* measures $z=0$. Since the only equation $0 \cdot s = 0$ is vacuously satisfied by any $s$, we need $n-1 = 0$ independent equations and immediately conclude $s=1$ (the unique non-zero 1-bit string). A measurement of $z=0$ is uninformative in isolation, but *guaranteed* - and the guarantee is itself the information.
- **$s=0$ (one-to-one):** the circuit samples $z \in \{0,1\}$ uniformly. Seeing $z=1$ means the equations reached full rank, proving $s=0^n$.

**Why it matters:** The $n=1$ case is small enough to trace by hand, making the edge case of "needing zero independent equations" crystal clear and motivating why the algorithm needs $n-1$ independent measurements in the general case.

---

### Section 7 · Case n = 2 (cell 17)

All three non-trivial secrets ($s \in \{01, 10, 11\}$) are demonstrated with `demo_simon`. With one qubit-pair, the allowed $z$-strings form a 1-dimensional subspace (two strings survive out of four), so a single non-trivial equation is sufficient. Each call prints the measured $z$'s, the true and recovered secrets, and a colour-coded bar chart.

---

### Section 8 · Case n = 3 (cell 18)

Four distinct secrets ($s \in \{001, 101, 110, 111\}$) are demonstrated. Now the surviving strings form a 2-dimensional plane (four out of eight strings survive), and two independent equations are needed. The plots dramatically illustrate that *exactly half* the strings are wiped out by destructive interference - a direct numerical confirmation of the theory in Section 2.

---

### Section 9 · The Qiskit Circuit (cells 19–22)

The same algorithm is now implemented as an actual quantum circuit using Qiskit 2.5.0 and the Aer simulator:

- `simon_oracle_circuit(s_bits, n)`: builds the 2n-qubit oracle gate. It first copies each input qubit to the corresponding output qubit (`CX(input_i, output_i)`), then picks the lowest set bit $j$ of $s$ as a control and adds `CX(input_j, output_k)` for each bit $k$ with $s_k = 1$. This folds every coset $\{x, x \oplus s\}$ onto the same output, enforcing the two-to-one promise.
- `build_simon_circuit(s_bits, n)`: wraps the oracle with Hadamard layers and measurements.
- `counts_to_z_arrays(counts, n)`: handles Qiskit's little-endian bitstring convention, reading `bitstr[-1-j]` as qubit $j$ to match the LSB-first arrays used throughout.

The circuit for $n=3$, $s=110_2$ is printed with `draw(output="text")`, and a histogram of shot counts for both $n=2$ and $n=3$ is compared to the NumPy marginal plots, confirming agreement. The secret is recovered correctly in both cases.

**Why it matters:** Bridging the "from scratch" NumPy implementation to the real Qiskit circuit shows that the matrix algebra in Section 4 is literally what the hardware (or simulator) executes. It also introduces students to Qiskit's endianness convention, a common source of bugs.

---

### Section 10 · Noise and Robustness (cell 23)

A depolarizing noise model is added: 2% single-qubit error on H and X gates, 5% two-qubit error on CX gates. With 8192 shots, roughly 8% of measurements land on *invalid* strings (those with $z \cdot s = 1$), which would inject incorrect equations into the linear system.

A **majority-vote recovery strategy** is then demonstrated: for each candidate secret $t$, compute the fraction of shots consistent with $t$ (i.e., $z \cdot t = 0$). The true $s$ scores near 1.0; any incorrect candidate scores near 0.5. Selecting the $t$ that maximises this score correctly recovers $s$ even under 8% noise.

**Why it matters:** Real quantum hardware is always noisy. Showing that a simple classical post-processing heuristic makes Simon's algorithm noise-tolerant without requiring quantum error correction is both practical and conceptually important.

---

### Section 11 · Exercises (cells 24–26)

Four progressively harder exercises, each with a deliberately broken placeholder (`return "TODO"` or `return -1`) that fails loudly:

- **E1 ★** Implement a consistency checker: return `True` iff all measured $z$'s satisfy $z \cdot s = 0 \pmod{2}$.
- **E2 ★★** Pick a random 4-qubit secret, build its oracle, run Simon, and verify the recovery.
- **E3 ★★★** Build an alternative valid oracle for a given secret (different label assignment) and confirm Simon still recovers $s$.
- **E4 ★★★★** Empirically estimate, over many trials, the average number of circuit runs needed before $n-1$ linearly independent equations are collected. (Theory predicts slightly above $n-1$; the exact expected value involves a harmonic-like sum over GF(2) subspace dimensions.)

A checker cell (`check_exercises`) runs all four and prints pass/fail. Collapsible `<details>` blocks contain full reference solutions.

**Why it matters:** Active problem-solving on the core operations - consistency checking, oracle construction, linear independence counting - turns passive reading into genuine understanding. The empirical exercise (E4) connects theory to simulation in a way that motivates the $O(n)$ claim with real numbers.

---

### Section 12 · Interactive Playground (cell 27)

`simon_playground(n, s)` accepts any $n \in \{1,2,3,4\}$ and any secret $s$, builds the oracle, runs Simon, and plots the colour-coded measurement distribution with the recovered secret in the title. When `ipywidgets` is available (Google Colab, live Jupyter), interactive sliders appear automatically. In static environments the function can be called directly with any arguments.

**Why it matters:** Experimentation cements intuition. Students who adjust $n$ and $s$ and watch the interference pattern change - always exactly half the strings surviving for any non-zero $s$, regardless of what $s$ is - develop a visceral sense of the algorithm's universality.

---

### Section 13 · Summary and What Comes Next (cell 28)

A closing markdown cell synthesises the key ideas, presents the classical vs. quantum query-complexity table, and explicitly warns against the "quantum parallelism" misconception: the superposition in Step 1 alone carries no information; the advantage arises only when the oracle correlates the registers and the second Hadamard triggers interference. The section ends by tracing the conceptual lineage to Shor's algorithm, positioning Simon as "the seed of Shor."

---

## Key Takeaways

- **Simon's algorithm achieves an exponential query speedup** ($O(n)$ vs. $\Omega(2^{n/2})$) by exploiting quantum interference, not classical parallelism - measuring the superposition right after Step 1 gives a useless random string.
- **The interference formula** $1 + (-1)^{s \cdot z} \in \{0, 2\}$ is the single algebraic fact that drives the entire algorithm: it destroys exactly those $z$ with $z \cdot s = 1$ and doubles the amplitude of those with $z \cdot s = 0$.
- **Indistinguishability causes interference.** Because $x$ and $x \oplus s$ produce identical oracle outputs, the quantum computer cannot tell which one it queried; their amplitudes must therefore be superposed and can cancel.
- **The quantum step only provides equations; a classical GF(2) solver finds $s$.** This is a general pattern in quantum algorithms - quantum hardware does the hard search, classical hardware does the interpretation.
- **Building oracles from scratch** (NumPy lookup tables, then Qiskit CX ladders) makes the algorithm concrete and debuggable; the `verify_function` guard ensures the two-to-one promise is never accidentally violated.
- **Noise injects wrong equations but not many.** A simple majority-vote consistency check over the measured shots is sufficient to recover $s$ reliably even at ~8% gate error rates without any quantum error correction.
- **Simon's problem is the hidden-subgroup problem over $(\mathbb{Z}_2)^n$**, and it is the conceptual prototype for Shor's algorithm (factoring) and the broad class of quantum algorithms based on the Quantum Fourier Transform.

---

## Further Reading & Citations

1. **Simon, D. R. (1997).** "On the Power of Quantum Computation." *SIAM Journal on Computing*, 26(5), 1474–1483.
   - The original paper introducing Simon's problem and the exponential separation. *(Conference version at FOCS 1994.)*

2. **Nielsen, M. A., & Chuang, I. L. (2010).** *Quantum Computation and Quantum Information* (10th anniversary ed.). Cambridge University Press.
   - Chapter 6 covers Simon's algorithm in full detail alongside Shor's and Grover's algorithms. The gold-standard textbook reference.

3. **Bernstein, E., & Vazirani, U. (1997).** "Quantum Complexity Theory." *SIAM Journal on Computing*, 26(5), 1411–1473.
   - Introduces the oracle model and query complexity framework that Simon's result builds on; defines the BQP complexity class.

4. **Shor, P. W. (1997).** "Polynomial-Time Algorithms for Prime Factorization and Discrete Logarithms on a Quantum Computer." *SIAM Journal on Computing*, 26(5), 1484–1509.
   *(arXiv: [quant-ph/9508027](https://arxiv.org/abs/quant-ph/9508027))*
   - Shows how Simon's hidden-subgroup idea generalises to $\mathbb{Z}_N$ via the QFT to give an efficient factoring algorithm.

5. **Childs, A. M. (2010).** "Lecture Notes on Quantum Algorithms." University of Waterloo.
   *(Available at: [https://www.cs.umd.edu/~amchilds/qa/](https://www.cs.umd.edu/~amchilds/qa/))*
   - Lecture 5 covers Simon's algorithm and the hidden-subgroup problem with careful complexity analysis and GF(2) linear algebra.

6. **Qiskit Community. (2024).** "Simon's Algorithm." *Qiskit Textbook / IBM Quantum Learning.*
   *(https://learning.quantum.ibm.com/)*
   - Official Qiskit implementation guide with circuit diagrams and explanations; continuously updated for the latest Qiskit API.

7. **Mosca, M. (2008).** "Quantum Algorithms." arXiv:0808.0369 [quant-ph].
   *(https://arxiv.org/abs/0808.0369)*
   - A broad survey placing Simon's algorithm within the full landscape of quantum algorithm design paradigms.

---

## Related Notebooks

All notebooks in the **Hands-On Quantum Computing** series (Trinexis):

| # | Title | Link |
|---|-------|-------|
| 01 | Introduction to Quantum Computing & Qubits | [Handson01](./Handson01_Intro_Qubits_Qiskit.ipynb) |
| 02 | Single-Qubit Gates and the Bloch Sphere | [Handson02](./Handson02_Single_Qubit_Gates_Qiskit.ipynb) |
| 03 | Multi-Qubit Systems and Entanglement | [Handson03_Entanglement_Qiskit.ipynb](./Handson03_Entanglement_Qiskit.ipynb) |
| 04 | Quantum Circuits and Measurement | [Handson04_Circuits_Measurement_Qiskit.ipynb](./Handson04_Circuits_Measurement_Qiskit.ipynb) |
| 05 | Superdense Coding and Quantum Teleportation | [Handson05_Teleportation_Qiskit.ipynb](./Handson05_Teleportation_Qiskit.ipynb) |
| 06 | Quantum Noise and Error Mitigation | [Handson06_Noise_ErrorMitigation_Qiskit.ipynb](./Handson06_Noise_ErrorMitigation_Qiskit.ipynb) |
| 07 | Deutsch–Jozsa Algorithm | [Handson07_DeutschJozsa_Qiskit.ipynb](./Handson07_DeutschJozsa_Qiskit.ipynb) |
| 08 | Bernstein–Vazirani Algorithm | [Handson08_BernsteinVazirani_Qiskit.ipynb](./Handson08_BernsteinVazirani_Qiskit.ipynb) |
| 09 | Grover's Search Algorithm | [Handson09_Grovers_Qiskit.ipynb](./Handson09_Grovers_Qiskit.ipynb) |
| 10 | Quantum Phase Estimation | [Handson10_PhaseEstimation_Qiskit.ipynb](./Handson10_PhaseEstimation_Qiskit.ipynb) |
| 11 | Quantum Fourier Transform | [Handson11_QFT_Qiskit.ipynb](./Handson11_QFT_Qiskit.ipynb) |
| 12 | Shor's Factoring Algorithm | [Handson12_Shors_Qiskit.ipynb](./Handson12_Shors_Qiskit.ipynb) |
| 13 | Variational Quantum Eigensolver (VQE) | [Handson13_VQE_Qiskit.ipynb](./Handson13_VQE_Qiskit.ipynb) |
| **14** | **Simon's Algorithm** *(this notebook)* | [Handson14_Simons_Algorithm_Qiskit.ipynb](./Handson14_Simons_Algorithm_Qiskit.ipynb) |
| 15 | Quantum Machine Learning Primer | [Handson15_QML_Qiskit.ipynb](./Handson15_QML_Qiskit.ipynb) |

---

<!--
SEO KEYWORDS:
simon's algorithm, simon's problem, quantum computing tutorial, qiskit tutorial, quantum speedup,
hidden subgroup problem, quantum oracle, quantum interference, destructive interference,
constructive interference, GF(2) linear algebra, gaussian elimination mod 2, quantum parallelism,
exponential quantum speedup, query complexity, birthday paradox quantum, quantum black box,
quantum circuit python, aer simulator, qiskit-aer, depolarizing noise, noisy quantum simulation,
quantum error mitigation, quantum post-processing, XOR period finding, bitwise XOR quantum,
quantum measurement, born rule, quantum entanglement oracle, hadamard gate, CNOT gate,
two-to-one function, one-to-one function, quantum algorithm lecture, Trinexis quantum,
Trinexis quantum computing education series, hands-on quantum, undergraduate quantum,
quantum computing course, quantum linear algebra, null space GF2, quantum Fourier transform,
Shor's algorithm precursor, quantum vs classical complexity, BQP complexity class,
quantum computing Python, numpy quantum simulation, quantum computing Jupyter notebook
-->
