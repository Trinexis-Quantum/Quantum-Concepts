# ContractBench: does a repair agent follow the contract that was in force?

**Subtitle:** A paired, execution-verified benchmark of 240 synthetic task pairs with dated ADR contracts, a tamper-resistant verifier and a paired success metric (PCSR). Track: Best New Resource.

## What it is

A patch can pass its tests and still violate the requirement that governed the code on the task date. In ContractBench every task comes as a pair: one codebase and one issue text, but two engineering contracts written as dated ADR documents, where a later ADR supersedes an earlier one. The task dated before the revision needs contract 0, and the task dated after it needs contract 1. A patch that follows the wrong contract can look fine and pass generic tests, so it is scored separately as a *wrong-contract* patch.

- 240 pairs, 480 tasks, 20 function families (5 train, 5 original test, 10 added test; `data/splits.json`). All synthetic and execution-validated: the legacy code fails both oracles, each reference patch passes its own oracle and fails the other's, and all pass the shared invariants.
- An independent verifier. It applies a patch to a fresh snapshot, overlays trusted oracle files after the patch so the agent cannot edit the grader, hashes them, and runs pytest in a scrubbed subprocess.
- Metrics: pass@1, wrong-contract rate, and PCSR (paired contract success rate: both sides of a pair solved for the same sample), with family-clustered bootstrap confidence intervals.
- Built-in controls: a time-agnostic index (`include_future=True`) and swapped-contract patches (`gold --swapped`). A scorer sanity check runs at both extremes: reference patches give PCSR 1.0, and swapped references give PCSR 0.0 with a wrong-contract rate of 1.0.
- Strict and lenient edit parsers, so format compliance and repair ability can be read separately (time-valid evidence arm: 0.428 strict, 0.789 lenient).

## Why it is new

Existing repair benchmarks show one repository state and ask whether a patch passes. ContractBench varies only the governing requirement while holding the code and the bug report fixed, so a model cannot succeed by matching the generic symptom. The paired design makes "followed the contract in force" measurable and comes with matched controls.

## What it shows (reference results, Gemma 4 12B, local)

Copied from `baselines/BASELINES.md`. The agent harness that produced them is described in the companion paper (submission 1).

| arm | what the agent gets | PCSR (lenient) | wrong-contract rate |
|---|---|---|---|
| B0 | tools only | 0.606 | 0.008 |
| B1 | + structural retrieval and call graph | 0.750 | 0.006 |
| **B2** | **+ time-valid decision records** | **0.789** | **0.000** |
| B2s | same records, time-agnostic index | 0.200 | 0.294 |
| B2x | same records, validity flags swapped | 0.111 | 0.403 |

| contrast | PCSR difference | 95% CI (family-clustered) | scope |
|---|---|---|---|
| time-valid evidence over time-agnostic index | +0.589 | [0.306, 0.844] | 60 pairs, 5 families |
| same | +0.621 | [0.469, 0.759] | 180 pairs, 15 families |
| time-valid evidence over swapped flags | +0.643 | [0.479, 0.790] | 180 pairs, 15 families |

The benchmark detects whether an agent uses validity markers: inverting the flags collapses PCSR by more than 0.6, while the patches stay well formed (about 98% apply). It separates an agent that reads the contract in force from one that writes plausible code.

## Verifiability

```
pip install -e .        # Python >= 3.10, pytest, pytest-timeout
python -m contractbench selfcheck       # reference patches resolve all tasks
python -m pytest tests                  # 9 tests, pass from a clean unzip
```

Apache 2.0, with `NOTICE` and `THIRD_PARTY_NOTICES.md`. The datacard documents composition, construction, scoring, intended use and privacy: the data is generated, and contains no personal data, credentials or third-party source code.

## Links

Companion paper: CoEvoGraph-SWE (submission 1).
