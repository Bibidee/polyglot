# Polyglot

**Polyglot is a standalone GenLayer Intelligent Contract for hash-bound translation review with GEN escrow.** A publisher commits a notice, specifies source and target languages, names a translator, and funds a reward. The translator commits the translated notice. Validators independently fetch and verify the exact source and translation bytes, then assess whether the translation preserves material meaning. Optional hash-bound images let vision-capable validators inspect source and translated scans alongside text.

This is a contract primitive, not a translation frontend or a claim that an AI can certify legal correctness. It is designed for notices where preserving meaning matters: appeal instructions, public-service notices, safety procedures, and policy communications.

## Why GenLayer?

A conventional contract can verify byte commitments and enforce deadlines, but cannot itself compare obligations across languages. A single off-chain service could make a selective or unauditable decision. Polyglot puts artifact retrieval and bounded semantic review inside GenLayer's independent validator process. The contract—not the model—derives the authorization outcome, controls the recipient, and accounts for the escrow.

Validators compare the decision outcome, not rationale wording. A payout is possible only if each valid result independently meets the entire safe tuple: both declared languages match, material meaning and each critical dimension are preserved, no material terms were added or omitted, risk is `no`, and confidence meets the threshold. Two validators agreeing on a non-approval can finalize a rejection or uncertainty even if they identify different reasons; neither path releases the translator reward. Approval-versus-non-approval disagreement fails equivalence.

The design reduces avoidable disagreement by comparing a small derived outcome rather than prose; it cannot guarantee that a transaction will finalize. Providers, web hosts, or validators can still fail or disagree, and GenLayer may leave a transaction undetermined. No state from an undetermined transaction is treated as authorization.

## Lifecycle

```text
Publisher validates task terms (nonpayable)
        |
        v
 awaiting_funding -- publisher cancels --> cancelled
        |
        +-- publisher allocates existing credit --> pending
                                                   |
                                                   +-- publisher cancels before acceptance --> refund dispatched
        |
        +-- named translator accepts --> accepted
                                      |
                                      +-- submits hash-bound translation --> submitted
                                                                              |
                                                                      semantic review
                                                         +--------------------+------------------+
                                                         |                    |                  |
                                                      approved             rejected          inconclusive
                                                         |                    |                  |
                                                translator payout       publisher refund    bounded review
                                                                                           or deadline dispute

submitted / retryable / inconclusive -- permanent evidence-integrity failure
                                      or unresolved review deadline --> disputed
                                                                       |
                                      both parties accept same split --+--> dispute_settled
```

Only the named translator can accept and submit. Only the publisher can cancel, and only before acceptance. Settlement is permissionless but uses the fixed translator or publisher stored in task state. The `deposited` ledger is zeroed and terminal state written before emitting the transfer. The payout amount comes from that ledger, never from model output or a caller argument.

## Semantic and evidence rules

- Language inputs use normalized language tags such as `en` and `es`, not display names such as `English` and `Spanish`.
- Source and translation artifacts are HTTPS URLs committed with normalized SHA-256 digests. The reviewer hashes exact raw response bytes before UTF-8 decoding.
- Optional source and translation images are each separately URL/hash-bound, size-limited, and restricted to PNG, JPEG, or WebP signatures. Up to two images are sent as untrusted visual evidence to the prompt.
- The prompt explicitly treats document text, images, and the publisher brief as untrusted data; instructions embedded in them must not be followed.
- Approval requires exact bounded fields for source/target language match, meaning, rights/duties, dates/amounts/conditions, exceptions, additions/omissions, risk, confidence, and rationale. Rationale is informational only.
- The model's `risk` field is defined as material risk introduced or concealed by a source/translation discrepancy, not risk inherent in the original notice and faithfully preserved in translation.
- A definitive semantic mismatch is `rejected` and permits publisher refund. Valid uncertainty is `inconclusive`; temporary web/provider/consensus failures are `retryable`. These outcomes do not authorize translator payout.
- Permanent evidence-integrity failures (hash mismatch, empty/oversized/invalid-UTF-8 artifacts, or invalid image content) become `disputed`, not synthetic semantic rejection. An unresolved submitted/retryable/inconclusive task also becomes `disputed` after its review deadline. Neither path pays automatically. Funds move only if publisher and translator both accept the exact same translator-share basis points. Otherwise escrow remains locked; the contract has no safe unilateral fault oracle or dispute timeout.
- Only the publisher or named translator can request review. A five-minute cooldown applies. The three-attempt limit counts valid semantic assessments only; transient observation/model/consensus failures do not consume those assessment attempts. The review deadline still bounds the review period, after which unresolved work is disputed.

## Escrow and transfer limits

Task creation is nonpayable. It validates and stores the agreed `reward_wei` in `awaiting_funding` before any payment. The publisher separately calls payable `deposit_credit()`; that minimal method records the attached GEN as credit for the actual sender. The publisher then calls nonpayable `fund_task(task_id)`, which allocates exactly the prepared task's reward from that publisher's existing credit and moves the task to `pending`. Overpayment remains the publisher's withdrawable credit. Invalid IDs, wrong callers, insufficient credit, and duplicate allocation fail in a nonpayable call and therefore cannot consume or misdirect funds. `withdraw_credit()` can withdraw only the caller's unused balance.

The `reward_wei` term remains separate from the actual `deposited` task liability. Aggregate accounting is `total_available_credits + total_locked_task_escrow`; `get_accounting()` reports that liability beside the current contract balance. Allocation moves liability between those ledgers without changing the total. Settlement and credit withdrawal clear their respective ledgers before emitting transfers. This prevents ordinary double-dispatch but cannot recover a downstream child transfer that fails after parent finalization.

`payout_dispatched` means the contract emitted a transfer message; it does not by itself prove the translator's wallet was credited. Integrators should verify the finalized child-transfer receipt and credited value where the network exposes them. A failed downstream transfer may not be recoverable by this contract.

## Security boundaries

- SHA-256 proves fetched-byte identity, not who authored a notice or whether its claims are true.
- HTTPS checks reject obvious malformed, local, and IP-literal targets. They cannot guarantee public DNS resolution, redirect safety, or domain ownership.
- Semantic review is not legal advice and cannot establish that the source notice is lawful or authoritative.
- Visual interpretation can miss text, layout, handwriting, or context. It is corroborative evidence, not OCR certification.
- Consensus can disagree or remain undetermined; only finalized approved state enables translator settlement.
- GEN transfer dispatch and recipient credit are separate facts.

## Build and release gate

Use Python 3.12 and the pinned tooling:

```bash
python -m pip install -r requirements.txt
python -m pytest tests/direct -q
python scripts/preflight.py
genvm-lint check contracts/polyglot.py --json
genvm-lint schema contracts/polyglot.py --output artifacts/polyglot.abi.json
```

Preflight requires exactly one deployable Python source, compiles source and tests, lints only `contracts/polyglot.py`, compares generated ABI with the tracked schema, and fails on test failures or skips. GitHub Actions runs this complete gate.

The contract header pins the GenLayer SDK dependency. Direct Mode is pinned to `genlayer-test==0.29.2`, stable validation runner `v0.2.16`, `genvm-linter==0.11.0`, and `pytest==9.1.1`. GenLayer's current developer guidance is available at [GenLayer Skills](https://skills.genlayer.com/), [Calling LLMs](https://docs.genlayer.com/developers/intelligent-contracts/features/calling-llms), [Image Processing](https://docs.genlayer.com/developers/intelligent-contracts/features/image-processing), [Web Access](https://docs.genlayer.com/developers/intelligent-contracts/features/web-access), [Value Transfers](https://docs.genlayer.com/developers/intelligent-contracts/features/value-transfers), and [The Equivalence Principle](https://docs.genlayer.com/developers/intelligent-contracts/equivalence-principle).

## Deployment

Polyglot v0.1.0 is deployed on Studionet at [`0x1E0239EBd297b29339Ec4cA767388C79EACc9078`](https://explorer-studio.genlayer.com/address/0x1E0239EBd297b29339Ec4cA767388C79EACc9078). Its frozen source hash is `498762a5c8b3426f21eb8499ee793e97d77d81a1b437430efb5378f85c3ba1d9`; deployed source parity is verified. A live `pending -> accepted -> submitted -> approved -> payout_dispatched` flow completed, and the child transfer receipt confirmed `0.001 GEN` credited to the designated translator. The working v0.1.1 source is **not deployed** and does not alter that address. It adds staged deposits, participant-only review, fair-dispute handling for unverifiable evidence, and custody accounting. This revision has 80 passing Direct Mode tests; lint, schema, and preflight pass locally. It has no v0.1.1 deployment, source-parity proof, hosted CI result, or live lifecycle. See [the deployment record](docs/DEPLOYMENT.md) for historical evidence and remaining limitations.
