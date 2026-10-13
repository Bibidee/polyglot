# Polyglot

**Polyglot is a standalone GenLayer Intelligent Contract for hash-bound semantic review of translations, with GEN escrow.** A publisher commits a notice, names a translator, and proposes the unresolved-task split. The named translator must accept those exact terms before any escrow is allocated. Validators then independently retrieve the exact committed source and translation bytes and assess whether rights, duties, dates, amounts, conditions, exceptions, and material meaning survive translation. Optional hash-bound images can provide visual context.

It is a reusable contract primitive, not a translation frontend, legal service, or claim that an AI can certify correctness.

## Why GenLayer?

A deterministic contract can enforce byte commitments and payouts but cannot itself compare legal or procedural meaning across languages. A single off-chain verifier can selectively return an approval without an independently reproduced judgment. Polyglot asks GenLayer validators to fetch the same committed artifacts and independently make the bounded semantic assessment; deterministic contract code alone derives the authorization and determines the fixed recipient.

Validators do not compare free-form rationale. For approval, both independently must satisfy the complete safe tuple. For rejection, which authorizes a publisher refund, validators must also agree on every bounded material classification field; different explanations of the same exact classification are acceptable, but different defect dimensions fail equivalence. `inconclusive`, transient failures, and evidence issues never authorize an immediate payout. The Equivalence Principle cannot guarantee that every transaction finalizes; disagreement or provider failures can still delay a task until its disclosed deadline allocation.

## Lifecycle

```text
publisher creates terms and selects fallback share (nonpayable)
  -> awaiting_acceptance
translator accepts exact terms
  -> awaiting_funding
publisher deposits GEN to own credit, then allocates exact reward
  -> pending
translator submits immutable translation/image hashes
  -> submitted
participant requests review
  -> approved -> payout dispatched to translator
  -> rejected  -> refund dispatched to publisher
  -> inconclusive / retryable / disputed -> re-review within limits
unsubmitted task reaches completion deadline, or unresolved review reaches review deadline
  -> fallback_settled using the creation-time share accepted before funding
```

An unfunded task can be cancelled without a transfer. A funded task cannot be unilaterally cancelled. Both parties may optionally agree to a different early split while the task is disputed. The deadline fallback is permissionless and cannot be changed by one party after acceptance. State and per-recipient intended transfer amounts are written before child transfers are emitted.

## Evidence and review rules

- Source and translation URLs are HTTPS and committed with SHA-256 hashes. Exact raw bytes are hashed before text decoding.
- Optional source and translation images have separate hash commitments, size bounds, and PNG/JPEG/WebP signature checks.
- The prompt treats text, images, and the publisher brief as untrusted data. Instructions inside artifacts are not reviewer instructions.
- Approval requires exact bounded classifications for language, meaning, rights/duties, dates/amounts/conditions, exceptions, additions/omissions, and risk, plus confidence of at least 80. Rationale never authorizes payment.
- A semantic rejection is consensus-critical because it permits a publisher refund: validators must agree on all bounded classification dimensions, not merely the word `rejected`. Rationale and confidence differences do not matter once the same semantic dimensions yield that same outcome.
- HTTP 404/408/410/425/429, redirects, 5xx, network exceptions, malformed model output, and consensus execution problems are non-authorizing and retryable. Other evidence integrity issues become `disputed`, but the original commitments remain fixed and the contract allows re-review during the recovery window.
- There are at most 12 finalized review calls total, at most 3 valid semantic assessments, exponentially increasing cooldowns (5 minutes to 6 hours), and a fixed review deadline. These bounds limit repeat cost and make the deadline behavior deterministic.

## Funding and unresolved-task economics

`create_task` is nonpayable and records the proposed `fallback_translator_bps` (0–10,000). The named translator must call `accept_task` before the publisher can fund the task. This acceptance covers the fallback allocation as well as task terms. The publisher then uses payable `deposit_credit()` to add credit only to the actual sender's ledger and nonpayable `fund_task()` to allocate the exact reward. Excess credit remains withdrawable by its owner. No caller can change the stored fallback share after task creation; a different early split requires both participants' explicit agreement.

This is an explicit allocation of evidence/timeout risk, not an adjudication of fault. The publisher chooses the proposed share and the translator can refuse it; both should set it deliberately before work. The contract cannot determine from a missing host or hash mismatch which participant caused the problem. If no semantic result is reached by the unsubmitted completion deadline or submitted review deadline, the accepted share is applied mechanically. This avoids indefinite escrow lockup but may not reflect fault or actual work performed; the participants must choose terms they consider acceptable.

The liability invariant is:

```text
total_accounted_liabilities = total_available_credits + total_locked_task_escrow
```

Settlement clears the task ledger and aggregate liability before dispatch. `get_accounting()` compares that ledger against the contract balance and exposes any shortfall or unallocated balance.

## Transfer reconciliation limitation

An emitted value transfer creates a child transaction: the amount leaves the sender and is held in the message before recipient credit; if the child fails, the value is not automatically returned. A split uses separate children, so one beneficiary can receive credit while the other does not, even though the task is terminal and both intended amounts are recorded. The contract cannot confirm downstream credit or recover a failed child itself. Integrators must compare each finalized parent transaction and task per-beneficiary amount with the corresponding exposed child receipt, `value_credited`, and recipient balance. Do not describe a `*_dispatched` or `*_settled` state as proof both beneficiaries were paid. See the official [GenLayer value-transfer documentation](https://docs.genlayer.com/developers/intelligent-contracts/features/value-transfers) and [message lifecycle documentation](https://docs.genlayer.com/developers/intelligent-contracts/features/messages).

## Security boundaries

SHA-256 proves byte identity, not source authorship or truth. URL admission cannot pin DNS resolution, prohibit all redirects, or establish host control. Semantic judgment is probabilistic, is not legal advice, and may remain undetermined. Images are evidence, not certified OCR. The payable credit method is intentionally minimal, but no contract can eliminate every platform/runtime failure mode. The v0.1.0 deployment and its historical child-transfer proof do not include the v0.1.2 candidate behavior.

## Build and verification

Use the pinned Python/GenLayer tooling:

```bash
python -m pip install -r requirements.txt
python -m pytest tests/direct -q
python scripts/preflight.py
genvm-lint check contracts/polyglot.py --json
genvm-lint schema contracts/polyglot.py --output artifacts/polyglot.abi.json
```

Preflight checks the one deployable contract, syntax, all Direct Mode tests, lint, and exact ABI match. The source pins its GenLayer dependency; requirements pin the test/linter stack.

## Deployment status

The only deployed version remains historical v0.1.0 at [`0x1E0239EBd297b29339Ec4cA767388C79EACc9078`](https://explorer-studio.genlayer.com/address/0x1E0239EBd297b29339Ec4cA767388C79EACc9078). The v0.1.2 branch is a new, undeployed candidate with a changed task-creation API and different escrow economics. It must not be represented as deployed or source-matched until an explicitly authorized Studionet deployment is finalized and parity is independently checked. Historical deployment and live evidence are retained in [docs/DEPLOYMENT.md](docs/DEPLOYMENT.md).

The previous v0.1.1 candidate's hosted CI run [38430058935](https://github.com/Bibidee/polyglot/actions/runs/38430058935) passed 80 tests with no failures or skips and passed lint, ABI, and preflight. The v0.1.2 candidate commit `083e53487e9baa6ce413a39a456eaf21ea4e3f4f` also passed hosted GitHub Actions run [38435878608](https://github.com/Bibidee/polyglot/actions/runs/38435878608). This CI result does not establish deployment, source parity, or live lifecycle evidence.

The current v0.1.2 candidate locally passes 99 Direct Mode tests (0 skipped, 0 failed), GenVM lint, ABI/schema generation, and preflight. Its contract SHA-256 is `9b439a833da0679fe45f810bcb4e626c6435f600159a3c3e7ba87e79f539682c`. These results do not imply hosted CI, deployment, or live lifecycle success; those remain unverified for v0.1.2.
