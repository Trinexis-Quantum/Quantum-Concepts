# ContractGate: a pre-execution check that a patch follows the contract in force

**Subtitle:** Time-valid ADR resolution, a code graph and a contract-conditioned world model that score a candidate patch as accept, flag or abstain before any test runs. Track: Best New Application.

## The use case

Teams revise requirements: a token lifetime moves from 45 seconds to 75, a discount rule changes. A patch for the old rule can look correct, pass generic tests, and still be wrong for a ticket filed after the revision. ContractGate scores a candidate patch against the contract in force *before tests are run*, using:

1. **Time-valid evidence.** Dated ADRs in the repository are parsed. Anything dated after the task date is invisible, and an ADR already superseded by an on-time revision is marked as such. The tool reports which ADR governs the patched function and which one it replaced.
2. **A code graph.** Callers and tests of the patched function give the blast radius.
3. **The C3-GWM world model.** A small gradient-boosted model predicts P(patch resolves the task) from the diff, the contract text and the graph counts.

It answers accept, flag or abstain (no dated ADR applies to that function, so it does not guess), and P(resolved) ranks candidate patches so the likeliest-correct one is tried first. A CLI provides `assess`, `demo`, `evaluate` and `train`.

Worked example (`python -m contractgate demo`, task dated after ADR-021, the `expiry` family held out of training): the fix that follows the contract in force gets ACCEPT with P(resolved) = 0.943. The fix that follows the superseded contract gets FLAG with P = 0.003 and the reason "introduces values that appear only in the superseded contract: ['45']".

## Results

Labels come from executing every candidate patch with a verifier; nothing is labelled by an LLM. Candidates are the own-contract reference fix, the other contract's reference fix, a no-op and up to 24 mutants. Intervals are 95%, family-clustered bootstrap.

| evaluation | model | AUPRC | pairwise accuracy (right contract's fix scored above the wrong one) |
|---|---|---|---|
| 10 original families, leave-families-out (6,480 patches) | contract-blind | 0.217 [0.146, 0.294] | 0.500 by construction |
| | **ContractGate** | **0.410 [0.265, 0.569]** | **0.779 [0.629, 0.929]** |
| | shuffled-contract control | 0.131 [0.092, 0.170] | 0.506 |
| 10 unseen families (5,798 patches) | contract-blind | 0.361 [0.284, 0.430] | 0.500 by construction |
| | **ContractGate** | **0.504 [0.384, 0.615]** | **0.758 [0.613, 0.896]** |
| | shuffled-contract control | 0.236 [0.188, 0.293] | 0.529 |

The contract-conditioned advantage over the contract-blind model (+0.143 [0.040, 0.249]) and over the shuffled control (+0.268 [0.167, 0.365]) replicates on families that no model or setting had seen. Handing the model the wrong contract removes the gain, so the score reflects the contract in force.

## Intended use

Rank candidate patches by P(resolved) before executing them, read the in-force and superseded ADR lines to see what a patch is judged against, and calibrate the accept threshold (`--threshold`) on your own data before using accept/flag as a hard decision. The tool complements running the tests and is not a security control. It parses the ContractBench ADR convention.

## Verifiability

```
pip install -e . && pip install -e ../contractbench
python -m contractgate demo
python -m contractgate evaluate --new-families      # ~20 s
python -m pytest tests                              # 10 tests, ~25 s
```

The packaged logic reproduces the paper pipeline's context for all 240 original tasks (720 feature vectors, 0 differences), and the new verifier agrees with the original sandbox labels on 240 randomly chosen patches (0 disagreements). Tests fail if AUPRC drifts more than 0.01 from the published values. Apache 2.0, with `NOTICE` and `THIRD_PARTY_NOTICES.md`; the shipped data contains no personal data or credentials.

## Links

Depends on ContractBench (submission 2). Companion paper: CoEvoGraph-SWE (submission 1).
