# Polyglot design

## Purpose and trust boundary

Polyglot is a standalone GenLayer Intelligent Contract for escrow-backed semantic review of a translation. It commits source and translated notices (and optional images) to exact SHA-256 values, then asks independent validators to evaluate whether important meaning, rights, duties, dates, amounts, conditions, exceptions, and scope survive translation.

Hashes prove byte identity, not authorship or truth. A publisher chooses the source URL and a proposed unresolved-task translator share; the named translator explicitly accepts those task terms before the publisher allocates escrow. Neither party can replace artifact commitments after task creation. Semantic review is not legal advice and external hosts, GenLayer web access, validators, and model providers remain trust boundaries.

## State machine

| State | Meaning | Legal next states and deadline behavior | Escrow |
|---|---|---|---|
| `awaiting_acceptance` | Valid nonpayable task terms recorded; translator has not accepted. | `awaiting_funding` on named translator acceptance before `accept_by`; `cancelled` by publisher or after acceptance deadline. | None |
| `awaiting_funding` | Translator accepted all recorded terms, including fallback share; publisher has not allocated reward. | `pending` after exact publisher-owned credit allocation before `accept_by`; `cancelled` by publisher or after deadline. | None |
| `pending` | Reward is funded, terms are accepted, and translation has not been submitted. | `submitted` by named translator before `completion_by`; `fallback_settled` at `completion_by`. | Full reward locked |
| `submitted` | Immutable translation commitments recorded; semantic review can begin. | `approved`, `rejected`, `inconclusive`, `retryable`, or `disputed`; unresolved state can be reviewed until `review_deadline`, then `fallback_settled`. | Full reward locked |
| `retryable` | Non-authorizing availability, model, or consensus failure. | Re-review same commitments after bounded cooldown; deadline -> `fallback_settled`. | Full reward locked |
| `inconclusive` | Valid semantic assessment cannot establish approval or definitive rejection. | Re-review while semantic attempts and review-call budget remain; deadline -> `fallback_settled`. | Full reward locked |
| `disputed` | A committed artifact failed integrity checks, or a participant proposed an early mutual split. This does not attribute fault. | Original immutable commitments may be re-reviewed after cooldown; mutual split can settle early; deadline -> `fallback_settled`. | Full reward locked |
| `approved` | Complete positive semantic tuple reached consensus. | Permissionless `settle` -> `payout_dispatched`. | Full reward locked until dispatch |
| `rejected` | Validators agree on the exact bounded material-defect classifications. | Permissionless `settle` -> `refund_dispatched`. | Full reward locked until dispatch |
| `payout_dispatched` / `refund_dispatched` | Terminal transfer message emitted to the stored translator/publisher. | Terminal; no blind retry. | Liability cleared before dispatch |
| `dispute_settled` | Both participants approved the same explicit early translator share. | Terminal; no blind retry. | Liability cleared before dispatch |
| `fallback_settled` | Unsubmitted completion deadline or unresolved review deadline applied the immutable creation-time share the translator accepted before funding. | Terminal; no blind retry. | Liability cleared before dispatch |
| `cancelled` | Unfunded task cancelled or expired before funding. | Terminal. | None |

At `now == deadline`, the preceding stage is closed: a translation/review cannot be submitted at its cutoff, and permissionless timeout settlement is available. A finalized state is needed; an undetermined transaction does not change canonical task state.

## Accepted fallback terms and incentive tradeoff

At task creation the publisher specifies `fallback_translator_bps` in `[0,10000]`. The named translator must accept the task, including that percentage, before any publisher credit can be allocated. The accepted value is immutable in task storage; a different early split is possible only when both participants separately approve it through the mutual-resolution methods.

At completion deadline, a funded task with no translation receives the creation-time split. At the review deadline, any unresolved submitted/retryable/inconclusive/disputed task receives the same split. The translator amount is floor(`deposited * bps / 10000`); the publisher receives the exact remainder, so intended transfers sum to the escrow exactly. This is a pre-agreed allocation of availability and timeout risk, not a model judgment or finding of fault. It prevents either party from holding funds forever and avoids an invented universal 50/50 rule. The publisher proposes the number; the translator may refuse before funding. The terms can still be unfair in a particular relationship, and the contract cannot know how much work was performed or who caused a host failure. Both parties must choose terms they knowingly accept.

## Evidence recovery and bounded review

The stored source and translation URL/hash pairs cannot be changed after task creation/submission. Each review re-fetches and verifies the original commitments. HTTP 404/408/410/425/429, redirects, 5xx responses, and fetch exceptions are operational unavailability and produce `retryable`; a 404 is not treated as proof of misconduct. Other HTTP rejections, invalid/empty/oversized bytes, invalid UTF-8, unsupported image signatures, or a hash mismatch produce a bounded `evidence_issue` that finalizes as `disputed`. `disputed` is still reviewable: if the original committed bytes become available again, semantic review can proceed without changing the hash. A mismatch is not automatically attributed to publisher or translator.

Review operations have a 5-minute initial cooldown doubling per finalized call up to 6 hours, a total cap of 12 finalized review calls, a maximum of 3 valid semantic assessments, and the stored review deadline. Operational/evidence failures do not consume the semantic-assessment counter, but they do consume the bounded review-call budget. When the budget is exhausted, no more reviews are accepted; timeout settlement remains available at the deadline. This creates a finite number of attempts and a deterministic end state, not an indefinite retry loop.

## Semantic result and validator equivalence

Artifacts and the publisher brief are untrusted quoted data. The model must not follow instructions inside them. For each valid structured result, deterministic code derives `approved`, `rejected`, or `inconclusive`:

- `approved`: languages match; meaning, rights/duties, dates/amounts/conditions, and exceptions are all preserved; no material term was added/omitted; `risk=no`; confidence >= 80.
- `rejected`: at least one definitive bounded defect (`no` on a required match/preservation dimension, `yes` for added/omitted terms, or `yes` for discrepancy risk).
- `inconclusive`: no definitive defect, but approval conditions are not all established.

Validators must agree on result kind and outcome. Approvals independently satisfy the full safe tuple. Because rejection permits a publisher refund, rejections additionally require exact agreement across the bounded classification dimensions: source/target language, meaning, rights/duties, dates/amounts/conditions, exceptions, additions/omissions, and discrepancy risk. Rationale and differing confidence do not matter if the exact same bounded classifications still derive rejection. Different classifications fail equivalence rather than turning a disagreement into a refund. `inconclusive` and operational failures are non-authorizing; their differing diagnostics cannot create immediate payout. Evidence failures must agree on stage and reason.

## Funding, accounting, and payment

The liability invariant is:

```text
total_accounted_liabilities = total_available_credits + total_locked_task_escrow
```

`create_task` is nonpayable. `deposit_credit` is the only payable task-funding entry point and credits `gl.message.sender_address`, not a caller-supplied owner. `fund_task` is nonpayable and allocates exactly `reward_wei` from that publisher's credit after translator acceptance. Excess deposits remain owner credit. `withdraw_credit` can clear only the caller's credit. Cancellation before funding creates no transfer.

For every task payout, refund, mutual split, or timeout split, the implementation reads from `task.deposited`, zeroes it, decrements `total_locked_task_escrow`, stores terminal status and the exact intended per-beneficiary amount, then invokes the one transfer helper. A second settlement finds no deposited amount and fails. The split remainder is assigned to the publisher, preventing rounding dust from remaining locked.

`get_accounting()` returns `available + locked`, current contract balance, unallocated balance, balance shortfall, and solvency. A child transfer can fail after the parent's liability is cleared; in that case the actual balance may exceed the recorded liabilities and appear as `unallocated_balance`. There is no supported callback or safe on-chain retry in this implementation to re-credit one failed child. Operators must reconcile by parent transaction, exposed child transaction, `value_credited`, intended amount, and recipient balance; do not retry blindly. The authoritative platform behavior is described in the [GenLayer value-transfer docs](https://docs.genlayer.com/developers/intelligent-contracts/features/value-transfers) and [message docs](https://docs.genlayer.com/developers/intelligent-contracts/features/messages): sent value resides in the message until child activation, and failed children are not automatically refunded.

## Public method authorization

| Method | Authorization and effect |
|---|---|
| `create_task` | Any publisher; task key uses actual sender; nonpayable; selects immutable proposed fallback share. |
| `accept_task` | Only named translator; accepts terms before funding. |
| `deposit_credit` | Any account; payable; credits only the actual sender. |
| `fund_task` | Only task publisher; allocates exact reward after translator acceptance. |
| `withdraw_credit` | Implicit caller; withdraws only caller's available credit. |
| `cancel_pending` | Publisher; only while task has no allocated escrow. |
| `submit_translation` | Named translator; commits fixed translation and optional image hashes. |
| `review` | Publisher or translator; review-call/semantic limits and deadlines apply. |
| `settle` | Permissionless; only `approved` pays fixed translator; only `rejected` refunds fixed publisher. |
| `timeout_refund` | Permissionless; cancels expired unfunded tasks, applies accepted split at completion/review deadline. |
| `propose_dispute_resolution` / `accept_dispute_resolution` | Either participant may propose in a disputed task; only the other can accept the exact share. |
| `get_task`, `get_info`, `get_accounting`, `get_available_credit` | Public views. |

## Limits

URL checks reject malformed HTTPS, credentials, nonstandard ports, local suffixes, and IP literals; they cannot guarantee DNS resolution, redirect behavior, or publisher control. Hashes do not prove source authenticity. Model output may be malformed or uncertain. Validator consensus may disagree or remain undetermined. Visual review is not certified OCR. No escrow result is legal advice. Child transfer state is dispatch status, not guaranteed credit.
