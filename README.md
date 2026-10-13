# Polyglot

**Polyglot is a standalone GenLayer Intelligent Contract for hash-bound semantic review of translations, with GEN escrow.** A publisher commits a notice, names a translator, and proposes the unresolved-task split. The named translator must accept those exact terms before any escrow is allocated. Validators then independently retrieve the exact committed source and translation bytes and assess whether rights, duties, dates, amounts, conditions, exceptions, and material meaning survive translation. Optional hash-bound images can provide visual context.

It is a reusable contract primitive, not a translation frontend, legal service, or claim that an AI can certify correctness.

## Why GenLayer?

A deterministic contract can enforce byte commitments and payouts but cannot itself compare legal or procedural meaning across languages. A single off-chain verifier can selectively return an approval without an independently reproduced judgment. Polyglot asks GenLayer validators to fetch the same committed artifacts and independently make the bounded semantic assessment; deterministic contract code alone derives the authorization and determines the fixed recipient.

Validators do not compare free-form rationale. Confidence means overall confidence in the complete classification set. Both approval and rejection require confidence of at least 80; anything lower is `inconclusive` and cannot directly authorize either settlement path. For rejection, which authorizes a publisher refund, validators must also agree on every bounded material classification field; different explanations of the same exact classification are acceptable, but different defect dimensions fail equivalence. `inconclusive`, transient failures, and evidence issues never authorize an immediate payout. A still-unresolved task may eventually reach the separate deadline allocation both parties accepted before funding. The Equivalence Principle cannot guarantee that every transaction finalizes; disagreement or provider failures can still delay a task until that disclosed allocation.

## Lifecycle

```text
publisher creates terms and selects separate non-submission and unresolved-review shares (nonpayable)
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
unsubmitted task reaches completion deadline
  -> fallback_settled using the non-submission share accepted before funding
unresolved submitted task reaches review deadline
  -> fallback_settled using the unresolved-review share accepted before funding
```

An unfunded task can be cancelled without a transfer. A funded task cannot be unilaterally cancelled. Both parties may optionally agree to a different early split while the task is disputed. The deadline fallback is permissionless and cannot be changed by one party after acceptance. State and per-recipient intended transfer amounts are written before child transfers are emitted.

## Evidence and review rules

- Source and translation URLs are HTTPS and committed with SHA-256 hashes. Exact raw bytes are hashed before text decoding.
- Optional source and translation images have separate hash commitments, size bounds, and PNG/JPEG/WebP signature checks.
- The prompt treats text, images, and the publisher brief as untrusted data. Instructions inside artifacts are not reviewer instructions.
- Approval requires exact bounded classifications for language, meaning, rights/duties, dates/amounts/conditions, exceptions, additions/omissions, and risk, plus confidence of at least 80. A definitive rejection also requires confidence of at least 80 and validator agreement on the bounded defect classifications. Below 80, every semantic tuple is `inconclusive`; rationale never authorizes payment.
- A semantic rejection is consensus-critical because it permits a publisher refund: validators must agree on all bounded classification dimensions, not merely the word `rejected`. Rationale and above-threshold confidence differences do not matter when the same semantic dimensions yield that same outcome.
- HTTP 404/408/410/425/429, redirects, 5xx, network exceptions, malformed model output, and consensus execution problems are non-authorizing and retryable. Other evidence integrity issues become `disputed`, but the original commitments remain fixed and the contract allows re-review during the recovery window.
- Review is bounded to at most 12 ordinary finalized calls plus one reserved recovery call. Up to 3 ordinary valid semantic assessments plus one reserved final assessment may occur. Operational and evidence failures use the ordinary call budget but do not consume the semantic-assessment count. The reserve can be used once after ordinary call exhaustion or after three semantic assessments, including during restored-evidence recovery near the deadline. It is never replenished. Cooldowns double from 5 minutes up to 6 hours, and the fixed review deadline remains an absolute cutoff.

## Funding and unresolved-task economics

`create_task` is nonpayable and records two independently chosen basis-point shares: `non_submission_translator_bps` and `unresolved_translator_bps`, each from 0 to 10,000. The first applies only when no translation is submitted by the completion deadline. The second applies only after submission when review remains unresolved at the review deadline. The named translator must call `accept_task` before the publisher can fund the task; that acceptance covers both shares and all task terms. The publisher then uses payable `deposit_credit()` to add credit only to the actual sender's ledger and nonpayable `fund_task()` to allocate the exact reward. Excess credit remains withdrawable by its owner. No caller can change either accepted share after task creation; a different early split requires both participants' explicit agreement.

These are explicit allocations of distinct deadline risks, not adjudications of fault. The publisher chooses both proposed shares and the translator can refuse them; both should set them deliberately before work. A non-submission share is not applied to a submitted task, and an unresolved-review share is not applied to a task that never reached submission. The contract cannot determine from a missing host, hash mismatch, provider failure, or validator disagreement which participant caused the problem or how much work was performed. If no semantic result is reached by the applicable deadline, that scenario's accepted share is applied mechanically. This avoids indefinite escrow lockup but may not reflect fault or actual work performed; participants must choose terms they consider acceptable.

The liability invariant is:

```text
total_accounted_liabilities = total_available_credits + total_locked_task_escrow
```

Settlement clears the task ledger and aggregate liability before dispatch. `get_accounting()` compares that ledger against the contract balance and exposes any shortfall or unallocated balance.

## Transfer reconciliation limitation

An emitted value transfer creates a child transaction: the amount leaves the sender and is held in the message before recipient credit; if the child fails, the value is not automatically returned. A split uses separate children, so one beneficiary can receive credit while the other does not, even though the task is terminal and both intended amounts are recorded. The contract cannot confirm downstream credit or recover a failed child itself. Integrators must compare each finalized parent transaction and task per-beneficiary amount with the corresponding exposed child receipt, `value_credited`, and recipient balance. Do not describe a `*_dispatched` or `*_settled` state as proof both beneficiaries were paid. See the official [GenLayer value-transfer documentation](https://docs.genlayer.com/developers/intelligent-contracts/features/value-transfers) and [message lifecycle documentation](https://docs.genlayer.com/developers/intelligent-contracts/features/messages).

## Security boundaries

SHA-256 proves byte identity, not source authorship or truth. URL admission cannot pin DNS resolution, prohibit all redirects, or establish host control. Semantic judgment is probabilistic, is not legal advice, and may remain undetermined. Images are evidence, not certified OCR. The payable credit method is intentionally minimal, but no contract can eliminate every platform/runtime failure mode. The v0.1.0 deployment and its historical child-transfer proof do not include the v0.1.3 candidate behavior.

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

The current submission deployment is **Polyglot v0.1.3** on Studionet: [`0x7B68e7E83BcBF6290c2Ef2C78663059ae0D131F8`](https://explorer-studio.genlayer.com/address/0x7B68e7E83BcBF6290c2Ef2C78663059ae0D131F8), deployed by [`0x8e580d0c2fd418c62bd2f0c8b5f0246b1c7b297a3348049cee2911530fea23e0`](https://explorer-studio.genlayer.com/tx/0x8e580d0c2fd418c62bd2f0c8b5f0246b1c7b297a3348049cee2911530fea23e0). Its 42,674-byte Explorer source matches the frozen repository contract byte-for-byte at SHA-256 `13a66b47654eb8168ef0a37bb8f94fd687c29b69b546702e3c231dedd9c0850f`. The finalized deployment reports `MAJORITY_AGREE` and GenVM `SUCCESS`; `get_info()` reports version `0.1.3`.

The v0.1.3 release-gate run [38442885261](https://github.com/Bibidee/polyglot/actions/runs/38442885261) completed successfully for frozen source commit `a2a14391c643e0344f79ec28992f837dabe9d6b2`: 161 Direct Mode tests passed, with 0 skipped and 0 failed; lint, schema, and preflight passed. The v0.1.1 run [38430058935](https://github.com/Bibidee/polyglot/actions/runs/38430058935) and v0.1.2 run [38435878608](https://github.com/Bibidee/polyglot/actions/runs/38435878608) remain historical candidate checks.

The v0.1.3 live lifecycle is also verified: task `PG-CODEX-LIVE-1791933532141` progressed through acceptance, minimum escrow funding, translation submission, semantic approval at confidence 98, and payout dispatch. The emitted child transfer finalized with `value_credited=true` for the designated translator. The final task state is `payout_dispatched`; full evidence and transaction links are in [docs/DEPLOYMENT.md](docs/DEPLOYMENT.md). The v0.1.0 deployment and lifecycle remain clearly labeled as historical there.
