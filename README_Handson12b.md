# Hands-on 12b — Quantum Oracles, Algorithm Primitives & Deutsch–Jozsa (Qiskit) 🔮

[![Qiskit](https://img.shields.io/badge/Qiskit-2.x-6929C4?logo=qiskit&logoColor=white)](https://qiskit.org/)
[![Qiskit Aer](https://img.shields.io/badge/Qiskit--Aer-0.17+-6929C4)](https://qiskit.org/ecosystem/aer/)
[![Python](https://img.shields.io/badge/Python-3.9%2B-3776AB?logo=python&logoColor=white)](https://www.python.org/)
[![Jupyter](https://img.shields.io/badge/Jupyter-Notebook-F37626?logo=jupyter&logoColor=white)](https://jupyter.org/)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](https://opensource.org/licenses/MIT)
[![Colab](https://img.shields.io/badge/Open%20in-Colab-F9AB00?logo=googlecolab&logoColor=white)](https://colab.research.google.com/)

> **FDP on Quantum Computing · IIT Roorkee · June–July 2026**
>
> *Unifying thesis: indistinguishable alternatives interfere; probabilities are squared magnitudes of amplitudes.*

---

## Overview

Imagine you have a mysterious black box — a function — and you need to answer a simple yes/no question about it: does it always return the same answer (constant), or does it return 0 for exactly half its inputs and 1 for the other half (balanced)? Classically, if you have a million-input function, you'd need to check over half a million inputs before you could be certain of the answer. A quantum computer, remarkably, answers this question with **a single query** to the black box — regardless of how many inputs there are. That is the Deutsch–Jozsa algorithm, and it is the first historically clean demonstration that quantum computers can be exponentially faster than their classical counterparts on a specific task.

This notebook is the Qiskit implementation companion to Notebook 12 (which built everything from scratch with NumPy arrays). Here you move from "here is the math" to "here is a real circuit running on a real simulator." You will see how quantum oracles are constructed as actual gate sequences, how the mysterious phenomenon of **phase kickback** works in practice, and how the full Deutsch–Jozsa circuit produces its remarkable result. Every key step is accompanied by a circuit diagram, a statevector check, and shot-based measurement output so you can see the quantum mechanics at work from multiple angles simultaneously.

The notebook is structured around a four-part template that recurs throughout quantum computing — state preparation, oracle query, interference, readout — and shows how Deutsch–Jozsa is the cleanest possible demonstration of this template. Understanding it deeply prepares you to read Bernstein–Vazirani, Simon's algorithm, Grover's search, and Shor's algorithm as natural variations on the same theme.

---

## Learning Objectives

After completing this notebook, you will be able to:

- Explain in plain language what a quantum oracle is, why reversibility forces us to use an ancilla qubit, and what the two standard oracle encodings (bit-flip and phase) do differently.
- Construct both a **bit-flip oracle** (`U_f`) and a **phase oracle** (`O_f`) as Qiskit `QuantumCircuit` objects for any classical Boolean function `f`.
- Derive and verify **phase kickback** numerically: show that running `U_f` with the ancilla in `|−⟩` is identical to running `O_f` on the data register alone.
- Build the **Deutsch** (1-bit) and **Deutsch–Jozsa** (n-bit) circuits in Qiskit, run them on `AerSimulator`, and interpret the measurement outcomes correctly.
- Navigate Qiskit's **little-endian bit-ordering convention** without getting confused by seemingly flipped bitstrings.
- Use `Statevector` for exact amplitude inspection and `AerSimulator` for shot-based sampling, and understand when each is appropriate.
- Articulate the precise classical vs. quantum query complexity comparison honestly, including the distinction between deterministic and randomised classical baselines.
- Recognise the four-part algorithm template (superposition → oracle → interference → readout) as the skeleton shared by all major quantum query algorithms.

---

## Background & Theory

### Classical Functions and the Problem of Reversibility

A classical Boolean function `f: {0,1}^n → {0,1}` is, in general, many-to-one: multiple inputs can produce the same output, so it cannot be directly inverted. But every quantum gate must be **unitary** — a reversible, norm-preserving transformation. You cannot simply build a gate that maps `|x⟩ → |f(x)⟩` because this would lose information whenever `f` is not a bijection.

The standard fix is to introduce an **ancilla qubit** `|y⟩` and define the oracle so that it acts on the pair `(x, y)`:

**Bit-flip (XOR) oracle:**
$$U_f \,|x\rangle|y\rangle \;=\; |x\rangle|y \oplus f(x)\rangle$$

This is reversible: applying `U_f` twice recovers the original state, because `(y ⊕ f(x)) ⊕ f(x) = y`. It is unitary. And it encodes `f` — set `y = 0` and the ancilla now holds `f(x)`. In a Qiskit circuit, this is implemented using `x` (NOT) gates to "select" a particular input pattern, then a multi-controlled-X (`mcx`) gate to flip the ancilla, followed by the same `x` gates to "uncompute" the selection.

### The Phase Oracle

The **phase oracle** takes a different approach: instead of writing `f(x)` into a separate register, it writes it into the *phase* of the state:

$$O_f \,|x\rangle \;=\; (-1)^{f(x)}\,|x\rangle$$

States where `f(x) = 0` are left alone; states where `f(x) = 1` are flipped to their negatives. This is a diagonal unitary matrix with `+1` and `−1` entries on the diagonal. In Qiskit, the same X-sandwich trick is used, but the multi-controlled operation is a phase gate (`mcp(π, ...)`) or a `Z` gate rather than an `X` gate.

### Phase Kickback: The Bridge Between the Two

Here is one of the most important — and most surprising — identities in quantum computing. Prepare the ancilla in the state `|−⟩ = (|0⟩ − |1⟩)/√2` and apply the bit-flip oracle:

$$U_f\,|x\rangle\,|{-}\rangle \;=\; (-1)^{f(x)}\,|x\rangle\,|{-}\rangle$$

Work through the algebra: `U_f` acts by XORing `f(x)` into the ancilla. When `f(x) = 0`, the ancilla is unchanged. When `f(x) = 1`, the ancilla flips from `|−⟩` to `−|−⟩` — but `−|−⟩ = −|−⟩`, so the minus sign "kicks back" into the phase of `|x⟩`. The ancilla itself is unaffected.

The result: **running `U_f` with the ancilla in `|−⟩` is exactly the same as running `O_f` on the data register**. This is why Deutsch–Jozsa (and every similar algorithm) prepares the ancilla as `H|1⟩ = |−⟩` at the start. You are given a bit-flip oracle from nature; phase kickback converts it into the phase oracle you need for interference to work.

> **Analogy.** Think of the ancilla in `|−⟩` as a "phase amplifier." When the oracle would normally leave a record in the ancilla, the `|−⟩` state instead absorbs the information and converts it into a global phase on the data qubit. The ancilla returns to its original state, leaving no trace — the information is now carried entirely in the relative phases of the data register.

### Uniform Superposition and Interference

Applying the Hadamard gate to all `n` data qubits creates a uniform superposition:

$$H^{\otimes n}\,|0\rangle^{\otimes n} \;=\; \frac{1}{\sqrt{2^n}}\sum_{x=0}^{2^n-1}|x\rangle$$

Every computational basis state is present with equal amplitude `1/√(2^n)`. This is sometimes (incorrectly) described as "evaluating `f` on all inputs simultaneously." It is not. This superposition, measured immediately, gives a uniformly random classical output — no better than a coin flip. The quantum advantage comes in the *next* step: the oracle attaches a phase pattern `(−1)^{f(x)}` to each term, and the *final Hadamard layer* then interferes these phases so that useful information (a global property of `f`) becomes readable in the measurement probabilities.

The Hadamard transform is its own inverse: `H·H = I`. Applied after the oracle, it implements the transformation:

$$H^{\otimes n}\,\frac{1}{\sqrt{2^n}}\sum_x (-1)^{f(x)}|x\rangle \;=\; \sum_z \alpha_z\,|z\rangle, \quad \alpha_z = \frac{1}{2^n}\sum_x (-1)^{f(x)+x\cdot z}$$

The amplitude at `z = 0...0` is particularly revealing:

$$\alpha_0 \;=\; \frac{1}{2^n}\sum_{x=0}^{2^n-1}(-1)^{f(x)}$$

If `f` is **constant**: all `(−1)^{f(x)}` have the same sign, so `|α_0| = 1`. The all-zeros state is measured with probability 1.

If `f` is **balanced**: exactly half the terms are `+1` and half are `−1`, so the sum cancels to zero. The all-zeros state has probability 0.

One query. Deterministic readout. Exponential classical advantage.

### Qiskit's Little-Endian Convention

Qiskit labels qubit 0 as the **least-significant bit** of the statevector index. If you have qubits `q[0], q[1], q[2]`, the state with `q[0]=1, q[1]=0, q[2]=1` is `|101⟩` in Qiskit's bitstring notation and corresponds to statevector index `0b101 = 5`. This is the **opposite** of the "MSB first" convention used in most textbooks. When you see a measurement outcome bitstring in Qiskit, the leftmost character corresponds to the highest-indexed qubit.

---

## Prerequisites

### Knowledge

- **Quantum mechanics basics:** bra-ket notation, state vectors, superposition, measurement (Notebook 3 of this series).
- **Quantum gates:** Hadamard (H), Pauli gates (X, Y, Z), controlled gates (CX, CZ, MCX) (Notebook 7).
- **Quantum circuits:** building and composing circuits, the circuit diagram metaphor (Notebook 8).
- **Qiskit basics:** `QuantumCircuit`, `transpile`, `AerSimulator`, `Statevector` (Notebook 9 and 11).
- **NumPy oracle version (recommended):** Notebook 12 (the NumPy companion) provides the mathematical scaffolding this notebook implements in hardware-targeting code.

### Software

```
Python >= 3.9
qiskit >= 2.0
qiskit-aer >= 0.17
numpy >= 1.24
matplotlib >= 3.6
pylatexenc          # for circuit diagram rendering
ipywidgets          # optional, for the interactive widget in Section 4.11
```

Install everything in one line (works on Google Colab):
```bash
pip install qiskit qiskit-aer pylatexenc ipywidgets
```

---

## Notebook Walkthrough

### Section 0 — Setup and Conventions

The notebook begins by installing and importing the required libraries, then printing version numbers to confirm the environment. A single shared `AerSimulator` instance (`SIM`) is created once and reused throughout.

**Why this matters:** Using a single shared simulator avoids re-initialising expensive backend objects and ensures all experiments use identical settings.

Section 0.1 states Qiskit's **little-endian convention** explicitly. This is easy to get wrong and causes subtle sign errors or seemingly flipped bitstrings. The notebook is upfront: qubit index 0 is the least-significant bit. The register layout for all oracles is fixed: `q[0]..q[n-1]` for data, `q[n]` for the ancilla.

Section 0.2 defines a compact **shared toolkit** of helper functions: `int_to_bits_lsb`, `bits_lsb_to_int`, `prepare_basis_state` (sets data register to `|x⟩` by flipping appropriate qubits), `statevector_of` (wraps `Statevector(qc).data`), `is_unitary` (checks unitarity via `U†U ≈ I`), `run_counts` (transpiles and runs on `AerSimulator`), and `marginal_probs` (converts shot counts to a probability vector). Building this toolkit once and reusing it throughout keeps subsequent code clean and focused on ideas rather than boilerplate.

### Section 1 — Quantum Algorithm Primitives

This section introduces the **four-role taxonomy** of quantum algorithm building blocks: state preparation, oracle query, phase manipulation, and interference/readout. The table maps each role to its concrete Qiskit implementation.

**Section 1.1: Uniform Superposition.** A three-qubit `H^⊗3 |000⟩` circuit is built, drawn, and verified: each of the 8 amplitudes equals `1/√8 ≈ 0.354`, and the total probability sums to 1. This sanity check establishes trust in the toolkit before using it for more complex circuits.

**The key warning in Section 1.1:** "Measuring `H^⊗n |0...0⟩` gives a uniformly random x — a classical fair coin." The uniform superposition is the *starting line*, not the finish line. Students often misread this step as the source of quantum speedup; the notebook addresses this misconception directly and early.

**Section 1.2: Interference Demo.** Three single-qubit circuits test `H·I·H`, `H·Z·H`, and `H·X·H`. The results — deterministically `|0⟩`, deterministically `|1⟩`, and still `|0⟩` respectively — demonstrate the core mechanism. The first H "opens two paths"; the second H "closes them," with constructive or destructive interference depending on what happened in between. This is a quantum Mach–Zehnder interferometer in one qubit.

### Section 2 — Quantum Oracles

**Section 2.1: Bit-flip oracle as a circuit.** `bitflip_oracle_circuit(f, n)` constructs `U_f` for any Python function `f` and qubit count `n`. The construction iterates over all `x` with `f(x) = 1`, X-sandwiches the data qubits where `x` has a `0` bit (so all data qubits read `1` and can trigger the MCX), applies `mcx`, then uncomputes the X-sandwich. This "X-sandwich + MCX" pattern is the canonical way to build multi-controlled unitaries conditioned on an arbitrary basis state.

The oracle is then demonstrated on `f(x) = x₀ ⊕ x₁` (parity on 2 bits) and verified to be unitary using `Operator(qc_Uf).data`. The action on all eight basis states `|x⟩|y⟩` is tabulated and compared against the expected `y ⊕ f(x)`.

**Section 2.2: Phase oracle as a circuit.** `phase_oracle_circuit(f, n)` uses the same X-sandwich structure, but the controlled operation is `Z` (for `n=1`) or `mcp(π, controls, target)` (for `n≥2`). A multi-controlled phase gate with angle `π` is a multi-controlled `Z`. The resulting `4×4` matrix for parity on 2 bits is printed: a diagonal matrix with entries `+1, −1, −1, +1` matching `(−1)^{f(x)}` for `x = 0, 1, 2, 3`.

**Section 2.3: Phase kickback — numerical verification.** For each basis state `|x⟩`, the notebook computes two statevectors: (LHS) run `U_f` with ancilla initialised to `|−⟩`, and (RHS) run `O_f` on the data register tensored with `|−⟩`. The Frobenius difference is at machine epsilon (`~1.2e-16`) for all four inputs, confirming the algebraic identity holds in practice.

**Section 2.4: Visualising the phase pattern.** A bar chart shows the amplitudes of `H^⊗3 |000⟩` before and after applying `O_f` for a chosen `f`. Bars for states with `f(x) = 1` flip from blue (`+1/√8`) to red (`−1/√8`). This is the fundamental picture: the oracle does not change probabilities (all bars have the same absolute height), it only sculpts the *signs*. The final Hadamard layer then reads those signs as outcome probabilities.

### Section 3 — Deutsch's Algorithm

The original 1985 Deutsch algorithm solves the simplest version of the problem: distinguish constant from balanced for `n = 1`.

**Section 3.1–3.2: Circuit and amplitude walk.** A 2-qubit circuit is built: `|0⟩|1⟩ → H⊗H → U_f → H → measure`. The four-step amplitude walk is written out symbolically, showing that after the final Hadamard the data qubit is exactly `|s⟩` where `s = f(0) ⊕ f(1)`. This is deterministic: no probabilistic outcomes, no error probability.

**Section 3.3: Testing all four one-bit functions.** The four functions (constant-0, constant-1, identity, NOT) are all run with 2048 shots. Every run gives 100% counts on the correct bit (`0` for constant, `1` for balanced). The output table shows this clearly. A bar chart compares the histograms for the two qualitatively different cases.

### Section 4 — Deutsch–Jozsa Algorithm

**Sections 4.1–4.2: Circuit and amplitude walk.** The `n`-bit generalisation is presented. The data register has `n` qubits; there is 1 ancilla. The promise is that `f` is either constant or balanced. The amplitude formula `α_z = (1/2^n) Σ_x (−1)^{f(x)+x·z}` is derived, and its value at `z = 0...0` is shown to be `±1` for constant and exactly `0` for balanced.

**Section 4.3: Implementation.** `deutsch_jozsa_circuit(f, n)` builds the circuit, draws it (a 4-qubit, deeply-layered circuit for `n=3`), and is ready for testing. The `if draw=True` parameter keeps code reuse clean.

**Sections 4.4–4.5: Constant and balanced cases.** For constant functions, all 2048 shots land on `'000'` — zero deviation. For balanced functions (including a fixed illustrative truth table and three random ones), `'000'` never appears. The contrast is absolute, not probabilistic.

**Section 4.6: Side-by-side histogram.** A `plt.subplots(1,2)` plot visualises the constant case (all mass on `'000'`, blue) against the balanced case (zero mass on `'000'`, red), making the distinction visually unmistakable.

**Section 4.7: Exact analysis with `Statevector`.** Shot-based counts have sampling noise. `dj_exact_probs` bypasses measurement entirely and computes the exact probability distribution by squaring amplitudes of the final statevector. For balanced functions, `P('000') = 0.000000` exactly — not approximately. This distinguishes simulation from real hardware and reinforces why shot-based and statevector simulators serve different pedagogical purposes.

**Section 4.8: Scaling to n = 4, 5.** Thirty random balanced truth tables are tested at each `n` from 2 to 5. The maximum `P('000')` over all trials is at machine epsilon (`~10^{-30}`). Deutsch–Jozsa's determinism is verified across all tested scales.

**Section 4.9: Classical vs. quantum query table.** A clean table prints `n`, `2^n`, the classical deterministic worst case `2^{n-1}+1`, and the quantum query count `1`. At `n = 20`, this is 524,289 vs. 1. The table is intentional: it makes the separation concrete and visceral.

**Section 4.10: `plot_histogram`.** Qiskit's native histogram plotter is demonstrated on the constant-vs-balanced comparison, introducing a tool students will use repeatedly in subsequent notebooks.

**Section 4.11: Interactive widget.** An `ipywidgets` slider and dropdown let students adjust `n` (1–4), choose function type (constant-0, constant-1, balanced auto, custom), and immediately see the exact probability distribution update. If `ipywidgets` is unavailable, a static fallback runs instead. This is the "explore on your own" moment of the notebook.

### Section 5 — Exercises

Four exercises with three difficulty tiers:

- **Exercise 1 (★):** Implement `constant_one_oracle(n)` without using `bitflip_oracle_circuit`. *Hint:* for `f ≡ 1`, just apply `X` to the ancilla unconditionally. Tests the student's understanding that the oracle's job is ultimately just to flip the ancilla based on `f(x)`.

- **Exercise 2 (★★):** Rewrite Deutsch–Jozsa using `phase_oracle_circuit` directly — no ancilla, a pure `n`-qubit circuit. This makes the connection between phase kickback and the phase oracle explicit: when you have the phase oracle directly, the ancilla machinery is unnecessary.

- **Exercise 3 (★★):** Compute the probability that a classical randomised algorithm (sampling `k` random inputs) is fooled by a balanced function. Derive the formula `2^{-(k-1)}` and plot it. Establishes that the *deterministic* classical complexity is `2^{n-1}+1` but *randomised* classical needs only `O(1)` queries for constant error — an important honesty check on the DJ narrative.

- **Exercise 4 (★★★):** What happens when the DJ promise is violated? For `f` with exactly `k` ones (not necessarily constant or balanced), derive `P('0...0') = ((2^n − 2k) / 2^n)^2` and verify numerically for `n = 4`, `k = 0,...,16`. The geometric interpretation (squared cosine of the angle between the sign vector and the all-plus vector) connects DJ to inner-product geometry and foreshadows the continuous optimisation perspective used in Grover analysis.

### Section 6 — Summary and Forward Pointers

A final section consolidates every concept built in the notebook, restates the four-part template, lists common misconceptions and their corrections, and suggests extensions including adding depolarising noise via `AerSimulator(noise_model=...)` as a bridge to Notebook 6's Kraus formalism.

---

## Key Takeaways

- **Reversibility forces the ancilla.** Classical functions are not invertible; quantum gates must be. The `(x, y) → (x, y ⊕ f(x))` construction is the standard workaround — it encodes `f` without destroying information.
- **Phase kickback converts bit-flip oracles into phase oracles.** Initialising the ancilla as `H|1⟩ = |−⟩` before applying `U_f` produces exactly the same effect on the data register as applying `O_f` directly. This is the core trick behind every query-based quantum algorithm.
- **Uniform superposition is the starting line, not the speedup.** Measuring it gives a fair coin. The speedup comes from using the oracle to mark phases, then interfering those phases so global properties of `f` become readable as measurement outcomes.
- **Deutsch–Jozsa completes in one oracle query regardless of n.** The classical deterministic worst case grows as `2^{n-1}+1`. This is the first clean exponential quantum advantage, even if the problem is artificial.
- **The all-zeros outcome is the detector.** For constant functions, all amplitude concentrates at `|0...0⟩` (constructive interference). For balanced functions, amplitude cancels to exactly zero at `|0...0⟩` (destructive interference). The measurement is binary and deterministic.
- **The four-part template — superposition, oracle, interference, readout — is universal.** Bernstein–Vazirani, Simon, Grover, QPE, and Shor are all specialisations of this skeleton. Understanding Deutsch–Jozsa deeply is understanding the skeleton.
- **Qiskit's little-endian convention requires active attention.** Bitstrings in measurement outputs have their bits ordered from highest to lowest qubit index (left to right), opposite to most textbooks. Forgetting this causes correctness errors that are hard to debug.

---

## Further Reading & Citations

1. **Deutsch, D. (1985).** "Quantum Theory, the Church–Turing Principle and the Universal Quantum Computer." *Proceedings of the Royal Society A*, 400(1818), 97–117. https://doi.org/10.1098/rspa.1985.0070 — *The original paper introducing the 1-bit Deutsch algorithm and the quantum Turing machine model.*

2. **Deutsch, D., & Jozsa, R. (1992).** "Rapid Solution of Problems by Quantum Computation." *Proceedings of the Royal Society A*, 439(1907), 553–558. https://doi.org/10.1098/rspa.1992.0167 — *The n-bit generalisation: the Deutsch–Jozsa algorithm and its exponential deterministic speedup.*

3. **Nielsen, M. A., & Chuang, I. L. (2010).** *Quantum Computation and Quantum Information* (10th Anniversary Ed.). Cambridge University Press. ISBN 978-1-107-00217-3. — *The standard graduate reference. Chapter 1.4 covers the Deutsch–Jozsa algorithm with full derivations; Chapter 6 covers oracles and query complexity.*

4. **Bernstein, E., & Vazirani, U. (1997).** "Quantum Complexity Theory." *SIAM Journal on Computing*, 26(5), 1411–1473. https://doi.org/10.1137/S0097539796300921 — *Introduces the Bernstein–Vazirani algorithm (the natural next step after DJ) and establishes the query complexity framework used to compare classical and quantum algorithms rigorously.*

5. **Cleve, R., Ekert, A., Macchiavello, C., & Mosca, M. (1998).** "Quantum Algorithms Revisited." *Proceedings of the Royal Society A*, 454(1969), 339–354. https://arxiv.org/abs/quant-ph/9708016 — *A unified treatment of Deutsch–Jozsa, Bernstein–Vazirani, and Simon's algorithm as instances of a single Fourier-sampling framework. Essential for understanding why the four-part template generalises.*

6. **IBM Qiskit Documentation — Circuit Library and Primitives.** https://docs.quantum.ibm.com/ — *The authoritative reference for `QuantumCircuit`, `AerSimulator`, `Statevector`, `Operator`, and all gates used in this notebook. The "Algorithms" section includes a Deutsch–Jozsa tutorial.*

7. **Aaronson, S. (2013).** *Quantum Computing Since Democritus*. Cambridge University Press. ISBN 978-0-521-19956-8. — *Chapter 10 gives a beautifully clear conceptual treatment of quantum query complexity, oracle separation, and what Deutsch–Jozsa actually proves (and does not prove) about computational advantage.*

---

<!-- SEO TAGS
quantum computing tutorial
Deutsch-Jozsa algorithm
quantum oracle Qiskit
phase kickback quantum
bit-flip oracle circuit
quantum algorithm primitives
quantum interference demo
Qiskit AerSimulator tutorial
quantum superposition Hadamard
Deutsch algorithm one qubit
quantum query complexity
constant vs balanced function
quantum speedup exponential
Qiskit QuantumCircuit oracle
multi-controlled gate mcx mcp
phase oracle diagonal unitary
quantum Fourier transform precursor
IIT Roorkee quantum FDP 2026
quantum computing education Jupyter
Bernstein-Vazirani precursor algorithm
-->

---

## Related Notebooks in This Series

| # | Notebook | Topic |
|---|----------|-------|
| 1–2 | [`Demo1-2_Double_Slit_and_Stern_Gerlach_FDP.ipynb`](Demo1-2_Double_Slit_and_Stern_Gerlach_FDP.ipynb) | Physical motivation: double-slit and Stern–Gerlach experiments |
| 3 | [`Demo3_QMPostulates_BraKet_Bloch.ipynb`](Demo3_QMPostulates_BraKet_Bloch.ipynb) | QM postulates, bra-ket notation, Bloch sphere |
| 4 | [`Demo4_BlochSphere_DensityMatrix.ipynb`](Demo4_BlochSphere_DensityMatrix.ipynb) | Density matrices and mixed states |
| 5 | [`Demo5_Purity_Coherence_Entanglement.ipynb`](Demo5_Purity_Coherence_Entanglement.ipynb) | Purity, coherence, and entanglement measures |
| 6 | [`Demo6_Noise_and_Information_Measures.ipynb`](Demo6_Noise_and_Information_Measures.ipynb) | Quantum noise channels and information theory |
| 7 | [`Demo7_Quantum_Gates_Demo.ipynb`](Demo7_Quantum_Gates_Demo.ipynb) | Universal gate sets and single-qubit operations |
| 8 | [`Demo8_QuantumCircuits_EntanglingGates_WHT.ipynb`](Demo8_QuantumCircuits_EntanglingGates_WHT.ipynb) | Entangling gates, circuits, and Walsh–Hadamard transform |
| 9 | [`Demo9_Qiskit_Introduction.ipynb`](Demo9_Qiskit_Introduction.ipynb) | Introduction to Qiskit 2.x |
| 10 | [`Demo10_PennyLane_Introduction_Hands_On.ipynb`](Demo10_PennyLane_Introduction_Hands_On.ipynb) | Introduction to PennyLane |
| 11 | [`Demo11_Qiskit_PennyLane_Synthesis_and_Comparison.ipynb`](Demo11_Qiskit_PennyLane_Synthesis_and_Comparison.ipynb) | Qiskit vs PennyLane synthesis and comparison |
| **12b** | **`Demo12b_Qiskit_Oracles_Primitives_DJ.ipynb`** ← *you are here* | **Oracles, primitives, and Deutsch–Jozsa (Qiskit)** |
| 13b | [`Demo13b_Bernstein_Vazirani_Qiskit.ipynb`](Demo13b_Bernstein_Vazirani_Qiskit.ipynb) | Bernstein–Vazirani algorithm |
| 14 | [`Demo14_Simons_Algorithm_Qiskit.ipynb`](Demo14_Simons_Algorithm_Qiskit.ipynb) | Simon's algorithm (exponential separation) |
| 15 | [`Demo15_Grover_Qiskit_FDP.ipynb`](Demo15_Grover_Qiskit_FDP.ipynb) | Grover's search algorithm |

---

*Prepared for the Faculty Development Programme on Quantum Computing, IIT Roorkee, June–July 2026.*
