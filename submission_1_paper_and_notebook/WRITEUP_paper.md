# CoEvoGraph-SWE: Co-Evolving Knowledge Graphs and Counterfactual World Models for Repository Repair

### Version-aware contract reasoning, competitive Gemma 4 agents, and execution-verified policy learning on NVIDIA DGX Spark

**Senthilkumar Vijayakumar, Sidharth E**

*Gemma 4 Developer Agent Paper Track. Every number below comes from `results_tables.md`, produced by the attached notebook (v2026-11-11) on one NVIDIA DGX Spark.*

## Abstract

A coding agent can edit the right function and still break the contract that governed that version of the repository. The difficulty is often less about finding the code than about knowing which requirement was authoritative when the code was written. CoEvoGraph-SWE is an offline-first Gemma 4 system built around that question. It combines a provenance-scoped development knowledge graph, version-gated external documentation, a Contract-Conditioned Counterfactual Graph World Model (C³-GWM), protected execution, and an evidence ledger that feeds supervised fine-tuning and execution-rewarded reinforcement learning.

We evaluate it with Paired Temporal-Contract Interventions: a pair counts only when both the historical and the revised version of a requirement are repaired correctly. On 240 synthetic pairs in 20 families, time-valid evidence reaches a Paired Contract Success Rate (PCSR) of 0.735 on 15 held-out families, against 0.114 for a time-agnostic index (clustered difference +0.621, 95% CI 0.469 to 0.759). Swapping the validity flags drops PCSR to 0.111 on the first held-out set, so the agent acts on the validity markers it is shown. A contract-conditioned predictor almost doubles the AUPRC for predicting patch resolution before execution, from 0.217 to 0.410 over 6,480 executed interventions, and a shuffled-contract control falls to 0.131. Verified-witness memory lifts the version-1 resolve rate from 0.853 to 0.907. A LoRA adapter trained on verified edits takes strict pass@1 from 0.614 to 0.864 and cuts the mean rollout wall time from about 115 s to 20 s, and one round of execution-rewarded RL raises PCSR over the DPO checkpoint by 0.278 (95% CI 0.044 to 0.600). Serving, verification, world-model fitting and LoRA training all run on a single DGX Spark.

## 1. Motivation and introduction

Consider two revisions of an authentication service. An earlier architecture decision, ADR-010 (2026-02-02), expires access tokens after 120 seconds. A later one, ADR-021 (2026-05-18), revises that to 40 seconds and makes the boundary exclusive. Both keep the invariant that signatures must always be verified. The bug report reads almost the same in both versions. A retrieval system that returns the newest or the most similar decision can produce valid Python that implements the wrong rule. The call graph tells you where the behavior lives, and the revision history tells you which behavior is correct. A patch needs both to be judged.

Repair benchmarks [1] show one repository state and ask whether a patch passes. Here the historical cutoff is part of the task definition, so the two contract versions are evaluated as one paired unit: the agent has to change its answer when the governing decision changes and keep the shared invariant while it does. We ask three questions.

- **RQ1.** Does version-valid evidence improve contract-correct repair?
- **RQ2.** Can a world model predict, before execution, which patch honours the contract in force?
- **RQ3.** Can verified outcomes flow back into the agent, through witness memory and a LoRA trained on verified edits, on the same device?

**Contributions.** (1) A version-scoped evidence pipeline and Paired Temporal-Contract Interventions: 240 synthetic pairs in 20 families plus 19 natural pairs from 10 repositories, scored by PCSR with family-clustered bootstrap intervals. (2) Matched controls that isolate what evidence contributes: a time-agnostic index and swapped validity flags. (3) C³-GWM, a contract-conditioned world model with a shuffled-contract control, judged by discrimination and by decision value. (4) An execution-verified loop on one DGX Spark: protected execution, a witness ledger, verified-witness memory, SFT and execution-rewarded RL.

## 2. Methods

**Model and testbed.** Gemma 4 12B-it [15], served by vLLM [14] with FP8 weights and an FP8 KV cache on one NVIDIA GB10, temperature 0.7, at most six turns. FP8 weights with an FP8 KV cache decode at 860.9 tokens/s against 452.3 for bf16 (1.90 times).

**Pairs.** A synthetic pair has one bug report and two dated decision records, the second superseding the first, plus same-vocabulary decoy records. Each contract has a reference implementation, contract-specific tests and shared invariant tests. Five families train. Five (bizdays, discount, email, password, slug; 60 pairs) form held-out set 1, and ten further families built on real standard-library behavior (`json.dumps`, `urllib.parse.quote`, `textwrap`, `shlex`, `Decimal` rounding and others; 120 pairs) form held-out set 2 and were never used for training or fitting. The 19 natural pairs are mined from git history: a later commit rewrites an assertion that an earlier commit introduced, validated by running the tests.

*Worked example (family `expiry`).* One bug report: `decide_expiry` does not follow the current token-lifetime contract. ADR-010 (2026-02-02): tokens expire after 45 s and are still valid at exactly 45 s. ADR-021 (2026-05-18) revises it: 75 s, and age 75 s is already expired. The version-0 task needs `elapsed > 45` and the version-1 task needs `elapsed >= 75`. A time-agnostic index shows version 0 the future ADR-021 and shows version 1 both records unmarked, so a single generic patch cannot satisfy both versions.

**Verifier.** Each candidate runs on a pristine `git archive` checkout, and trusted oracle files are overlaid after the patch and hashed. A task is *resolved* if the patch applies and passes its contract's tests and the invariants. *Wrong-contract* means it fails its own tests while passing the other version's. PCSR counts a pair at sample *s* if both versions resolve at *s*. Intervals resample pairs clustered by family (4,000 draws), and a contrast counts as a finding when its interval excludes 0 and the paired test agrees. The verifier resolves 480 of 480 synthetic and 38 of 38 natural gold patches and credits 0 empty patches; 142 pytest checks pass; a replay of 200 stored rollouts reproduces every outcome and test flag.

**Scorers.** A strict parser requires `<<<<<<< SEARCH <path>`. A lenient parser accepts other path forms and infers a missing path only if the search text occurs in exactly one non-test file; sandbox and oracles are unchanged. Lenient measures repair ability and strict measures format compliance, and both are reported.

**Arms.** One prompt and SEARCH/VIEW tool loop for all arms; only evidence differs. B0: tools only. B1: plus BM25 symbols, a one-hop call graph and tests. B2: plus time-valid decision records (cutoff filter, superseded records flagged). B2s: B2 from a time-agnostic index. B2x: B2 with in-force and superseded flags swapped.

**Leakage audit.** Over 480 tasks, time-valid retrieval exposes 0 post-cutoff documents, and the time-agnostic index exposes the revising ADR in 240 of 240 version-0 tasks.

## 3. Results

### 3.1 RQ1: the agent follows the contract in force

| Arm | PCSR lenient [95% CI] | PCSR strict | wrong-contract | applied |
|---|---|---|---|---|
| B0 | 0.606 [0.367, 0.783] | 0.183 | 0.008 | 0.897 |
| B1 | 0.750 [0.594, 0.889] | 0.294 | 0.006 | 0.972 |
| B2 | 0.789 [0.422, 0.983] | 0.428 | 0.000 | 0.989 |
| B2s | 0.200 [0.028, 0.361] | 0.122 | 0.294 | 0.981 |
| B2x | 0.111 [0.017, 0.267] | 0.094 | 0.403 | 0.983 |

*Held-out set 1, 360 rollouts per arm.* B2 minus B2s is +0.589 [0.306, 0.844] on set 1 and +0.621 [0.469, 0.759] on all 15 families. B2 minus B2x is +0.678 [0.333, 0.917] and +0.643 [0.479, 0.790]. B2 minus B0 is +0.183 [0.011, 0.333] on set 1. B2s and B2x apply as often as B2, so the agent writes well-formed patches for the wrong contract: B2x holds the same documents as B2 with only the markers inverted, and 40% of its patches implement the other version. Pooled over 15 families, B2 reaches PCSR 0.735 and B2s 0.114.

### 3.2 RQ2: a contract-conditioned world model

**Data.** 6,480 executed interventions on 240 tasks (own reference, other-contract reference, no-op, 24 AST mutants per task), with sandboxed per-test outcomes as labels; 7.5% resolve. Models predict `resolved` before execution. B5: static reachability. B6: logistic regression on structural features. B7: gradient-boosted trees on structure plus hashed patch tokens. B8 (C³-GWM): B7 plus features comparing the patch with the contract in force (contract constants realised in the code, new constants unjustified by it, constants found only in the other contract). Evaluation is family-disjoint 5-fold cross-validation; B8-shuffled receives another task's contract.

| Model | AUPRC [95% CI] | Pool selection, K=8 |
|---|---|---|
| B5 | 0.149 [0.112, 0.190] | 0.259 |
| B6 | 0.205 [0.142, 0.270] | 0.374 |
| B7 | 0.217 [0.144, 0.294] | 0.356 |
| **B8** | **0.410 [0.267, 0.570]** | **0.501** |
| B8-shuffled | 0.131 [0.090, 0.172] | 0.238 |

B8 exceeds B7 by +0.194 [0.116, 0.284] and the shuffled control by +0.279 [0.152, 0.426], so the gain comes from the contract in force. Choosing one of eight executed candidates, B8 finds a resolving patch 0.501 of the time against 0.157 for a random pick, and beats B7 by +0.129 [0.054, 0.220]. A graph-attention variant benefits from conditioning too (+0.116 [0.019, 0.220] over the same network without contract nodes), with the gradient-boosted B8 as the strongest model. On 1,440 real Gemma 4 patches, ranking quality among the patches that applied is AUROC 0.761 for B8 against 0.615 for B7.

### 3.3 RQ3: verified outcomes flowing back

**Witness memory.** B10 adds a verified version-0 diff, flagged as superseded, to the version-1 prompt. It resolves 0.907 of rollouts against 0.880 for the same diff unflagged and 0.853 for B2 (150 rollouts each; B10 minus B2 +0.064 [0.000, 0.181]). The headroom sits in the `slug` family: 0.417 for B10 against 0.125 for B2.

**Adaptation on one device.** One LoRA [11] (r=16, all language-model linear layers) trains on the five training families. T1 is verified SFT, T2L adds DPO [12] and T3aL adds one round of GRPO-style [13] execution-rewarded RL, both on clean lenient-parser labels. Reward: 1.0 resolved + 0.25 own-contract pass + 0.10 invariants minus a length cost. Evaluation is held-out set 1 (60 pairs, 3 samples).

| Model | PCSR | strict pass@1 | applied | turns | mean wall s / rollout |
|---|---|---|---|---|---|
| T0 (prompting) | 0.789 | 0.614 | 0.989 | 3.74 | 114.7 |
| T1 (SFT) | 0.767 | 0.864 | 1.000 | 1.00 | 20.0 |
| T2L (+DPO) | 0.478 | | 0.819 | 2.21 | 58.6 |
| T3aL (+1 RL round) | 0.756 | 0.811 | 1.000 | 1.22 | 21.8 |

SFT teaches the exact edit format: every edit applies, strict pass@1 rises from 0.614 to 0.864, and a rollout takes one turn and about 20 s instead of 3.7 turns and 115 s, at PCSR in the range of untuned prompting. One RL round lifts PCSR over the DPO checkpoint from 0.478 to 0.756 (+0.278 [0.044, 0.600]; 16 pairs gained, none lost) with every edit valid. SFT takes 19 steps (627 s, 33.2 GB peak) and the RL round 65 steps (2,154 s), all on the same GB10 that serves the model.

## 4. Related work

**Repository graphs and agents.** RepoGraph [2] adds a repository-level code graph for navigation, and the Code Graph Model [3] embeds graph structure into attention, reporting a 43.00% SWE-bench Lite resolution rate with Qwen2.5-72B. Our graph is a plain call, test and document graph, and the B1 arm shows the usual direction (+0.144 PCSR over B0).

**World models for software.** Software World Models [4] predict which components break after a change and argue for judging prediction by decision value, and COUNTERMEM [5] builds memory by checking counterfactual actions with executable world models. We add conditioning on a dated contract with a shuffled-contract control, and measure decision value by candidate selection.

**Implicit requirements.** SWE-RPG [6] reports implicit requirement recovery as the main bottleneck (24.5% to 46.0% of agent runs), and SWE-Proof [7] finds that a quarter of test-passing patches admit counterexamples. We isolate one source of implicit requirement, temporal validity, with paired controls. We did not find prior work pairing one bug report with two dated requirements, but did not search exhaustively.

**Training SWE agents.** Self-play SWE-RL [8], Anchored Self-Play [9] and the Agentic Verifier [10] train and verify at far larger scale. We use LoRA, DPO and GRPO-style updates in their standard form at a scale that fits one DGX Spark.

## 5. Verifiability and conclusion

**Verifiability.** One notebook regenerates every table from stored rollouts (about 10 to 15 minutes including the live GPU checks) or reruns experiments on a GB10, and a manifest records versions (torch 2.13.0, vLLM 0.32.0, transformers 5.19.0, PEFT 0.21.2) and SHA-256 hashes of adapters, tasks and verifier code. 142 pytest checks cover the temporal cutoff, reward integrity, split disjointness, the sandbox and the documentation gate. Code and the notebook are Apache 2.0 with third-party notices; the notebook contains no credentials, personal data or machine-specific paths.

**Conclusion.** For a local 12B agent, the clearest lever on contract-correct repair is whether retrieved evidence carries correct validity markers, shown by controls that change nothing else. A contract-conditioned world model sees which patches honour that contract before they run, and the same device that serves the model also verifies, predicts and trains. The released ContractBench (submission 2) and ContractGate (submission 3) package the benchmark and the scorer for reuse.

## References

[1] Jimenez et al. SWE-bench: Can language models resolve real-world GitHub issues? ICLR 2024.
[2] Ouyang et al. RepoGraph: Enhancing AI software engineering with repository-level code graph. ICLR 2025.
[3] Tao et al. Code Graph Model (CGM): a graph-integrated LLM for repository-level software engineering tasks. NeurIPS 2025.
[4] Su et al. Software World Models: from consequence prediction to decision value. arXiv:2610.04940, 2026.
[5] Pu, Tang, Zhang. COUNTERMEM: world-model verified counter-factual memory for language agents. arXiv:2609.31874, 2026.
[6] Zhou et al. A unified issue resolution benchmark for requirement clarification, planning, and code generation (SWE-RPG). arXiv:2608.09072, 2026.
[7] Ma et al. SWE-Proof: can language models resolve real-world issues with machine-checked proofs? arXiv:2609.21190, 2026.
[8] Wei et al. Toward training superintelligent software agents through self-play SWE-RL. ICML 2026 (PMLR 306).
[9] Choi et al. Anchored self-play for code repair. ICML 2026 (PMLR 306).
[10] Ma et al. Scaling agentic verifier for competitive coding. ICML 2026 (PMLR 306).
[11] Hu et al. LoRA: low-rank adaptation of large language models. ICLR 2022.
[12] Rafailov et al. Direct preference optimization. NeurIPS 2023.
[13] Shao et al. DeepSeekMath (introduces GRPO). arXiv:2402.03300, 2024.
[14] Kwon et al. Efficient memory management for LLM serving with PagedAttention (vLLM). SOSP 2023.
[15] Google DeepMind. Gemma 4 technical report. arXiv:2607.02770, 2026.

*Code, generators, notebook and tables are attached to this writeup.*
