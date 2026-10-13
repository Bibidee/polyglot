# Polyglot design

## Purpose and trust boundary

Polyglot is a standalone GenLayer Intelligent Contract for escrow-backed semantic review of a translation. It binds source and translated notices (and optionally their images) to exact SHA-256 commitments, then asks independent validators to assess whether important meaning, rights, duties, dates, amounts, conditions, and exceptions survive translation.

Hashes prove byte identity, not authorship or truth. Language and semantic judgments are not legal advice. Publisher, translator, external HTTPS hosts, GenLayer web access, and the LLM are all trust boundaries; only deterministic contract rules select amounts, recipients, and state transitions.

## State machine

| State | Meaning | Legal next states |
|---|---|---|
| `awaiting_funding` | Validated task terms recorded; no escrow yet. | `pending`, `cancelled` |
| `pending` | Exact reward allocated to task. | `accepted`, `refund_dispatched` |
| `accepted` | Named translator accepted. | `submitted`, `refund_dispatched` |
| `submitted` | Translation commitment stored and review can begin. | `approved`, `rejected`, `inconclusive`, `retryable`, `disputed` |
| `retryable` | Temporary observation/provider/consensus failure. | `approved`, `rejected`, `inconclusive`, `retryable`, `disputed` |
| `inconclusive` | Valid semantic analysis cannot establish safe parity or definitive mismatch. | `approved`, `rejected`, `inconclusive`, `retryable`, `disputed` |
| `approved` | Complete positive semantic tuple reached consensus. | `payout_dispatched` |
| `rejected` | Definitive material translation defect reached consensus. | `refund_dispatched` |
| `disputed` | Evidence integrity failed or review deadline elapsed; no safe automatic fault assignment. | `dispute_settled` only after exact mutual split agreement |
| `payout_dispatched` | Transfer emitted to stored translator; child credit not implied. | Terminal |
| `refund_dispatched` | Refund emitted to stored publisher; child credit not implied. | Terminal |
| `dispute_settled` | Both participants approved the same split; transfers emitted. | Terminal |
| `cancelled` | Unfunded prepared task cancelled. | Terminal |

## Public method authorization

| Method | Who can call | Value effect |
|---|---|---|
| `create_task` | Any publisher, task namespaced by actual sender | Nonpayable; records validated terms |
| `deposit_credit` | Any account | Payable; adds credit only to actual sender |
| `fund_task` | The task's publisher | Nonpayable; moves publisher credit to task escrow |
| `withdraw_credit` | Credit owner (implicit caller) | Clears only caller's credit, then dispatches transfer |
| `cancel_pending` | Stored publisher | Cancels unfunded task or refunds funded `pending` task before acceptance |
| `accept_task` / `submit_translation` | Stored translator | Changes only its own task's workflow state |
| `review` | Stored publisher or translator | Requests nondeterministic review; no direct money movement |
| `settle` | Permissionless | Pays only fixed translator after `approved`, or publisher after `rejected` |
| `timeout_refund` | Permissionless | Refunds only pre-submission timeout cases; unresolved post-submission review becomes `disputed` |
| `propose_dispute_resolution` | Publisher or translator | Records proposed translator share in basis points |
| `accept_dispute_resolution` | The other participant | Dispatches only if it accepts the exact proposed share |
| `get_task`, `get_info`, `get_accounting`, `get_available_credit` | Public views | Read-only |

Task identifiers are namespaced by publisher address. Address parameters used for reads/operations are normalized and task participant checks use stored addresses.

## Funding and accounting invariant

Task terms are created in nonpayable `create_task(...)`, which stores `reward_wei` and `awaiting_funding`. A separate minimal payable `deposit_credit()` records `gl.message.value` against the actual sender's address, without parsing task IDs, URLs, hashes, language tags, or other task input. The task publisher then calls nonpayable `fund_task(task_id)`, which moves exactly `reward_wei` from that publisher's available credit to the task's `deposited` ledger. Any excess remains the publisher's available credit.

The explicit liability invariant is:

```text
accounted liabilities = total_available_credits + total_locked_task_escrow
```

Depositing increases available credit and liabilities by the same amount. Funding moves value between available and locked ledgers without changing total liabilities. Cancellation, refund, payout, or mutually agreed dispute settlement first zeroes the relevant user/task ledger and decrements aggregate liabilities, then emits the transfer. This prevents ordinary double withdrawal/settlement and prevents one publisher's credit from funding another publisher's task.

`get_accounting()` compares recorded liabilities with `self.balance` at read time. Outgoing transfers are asynchronous child messages: the parent can finalize with a transfer dispatched while the child later fails or remains unresolved. The contract has no callback that safely restores a failed child amount, so it does not blindly retry. Operators must inspect the finalized child receipt and `value_credited`; a dispatched status alone is not a successful payment claim.

The v0.1.0 deployment's invalid payable task-creation transaction is historical: it credited 0.001 GEN while task execution failed, leaving no old-version recovery record. v0.1.0 has no recovery method; v0.1.1 cannot recover value held by that separate address.

## Evidence review and consensus

Before nondeterministic execution, the contract copies task fields into a plain local snapshot. Leader and validators independently fetch the same committed source/translation bytes and optional image bytes. Text artifacts must be nonempty, within the byte bound, exact SHA-256 matches, and valid UTF-8. Images must be bounded, hash-matching PNG/JPEG/WebP signature data. The web client may follow redirects or resolve hostnames in ways this contract cannot constrain; `_host()` rejects malformed HTTPS and obvious local/IP-literal targets but is not DNS pinning or redirect protection.

The model receives the untrusted source, translation, brief, and optional images with an instruction not to follow embedded instructions. The `risk` field means material risk introduced or concealed by a source/translation discrepancy; risk inherent in the source and faithfully preserved is not itself a translation defect. Output must match the exact bounded schema. Deterministic code derives approval only if every language/meaning/rights/duties/date/amount/condition/exception check is positive, additions/omissions and risk are negative, and confidence reaches the threshold. Rationale cannot affect authorization.

Equivalence compares the bounded authorization outcome, not free-form rationale. Approval is equivalent only when both outputs independently derive the full approved tuple. Different safe rejection/uncertainty reasons may agree because neither authorizes translator payment. Approval-versus-nonapproval disagreement fails consensus. Failures with distinct operational details are both nonauthorizing; evidence-integrity results must agree on the same stage and reason to become a recorded dispute.

## Failure policy and fairness

| Observation | Result | Funds |
|---|---|---|
| Temporary fetch/network/provider/consensus failure | `retryable` | Remain escrowed; no payout/refund |
| Malformed model output | `retryable` | Remain escrowed; no payout/refund |
| Valid clear semantic mismatch | `rejected` | Publisher refund may be dispatched |
| Valid ambiguous/uncertain assessment | `inconclusive` | Remain escrowed; may be reviewed again within limits |
| Permanent source/translation/image integrity failure | `disputed` | Frozen; not automatically attributed to either party |
| Submitted/retryable/inconclusive task passes review deadline | `disputed` | Frozen; no automatic timeout refund |

Only publisher/translator may request review. A five-minute cooldown applies. `attempts` counts valid semantic assessments only (including `inconclusive`); transient failures, malformed output, or consensus execution failure do not consume the semantic attempt budget. Valid reviews are limited to three and the review deadline bounds the period.

Disputed escrow moves only when one participant proposes a 0–10,000 basis-point translator share and the other participant accepts that exact value. The remainder goes to the publisher. This is a mutual settlement mechanism, not adjudication. If the parties never agree, escrow can remain locked indefinitely. A unilateral fallback would invent fault/economic outcomes not provable from available evidence, so this version deliberately does not provide one.

## Images and prompt-injection limits

Images are optional corroborating visual evidence, not guaranteed OCR or legal interpretation. Direct Mode tests use complete small PNG bytes and mock the model response; they do not prove deployed provider vision behavior. Artifact text/images can still probabilistically influence a model despite explicit prompt boundaries. Hashing preserves submitted bytes but cannot make them benign.

## Current verification boundary

The working candidate v0.1.1 locally passes 80 Direct Mode tests, GenVM lint, generated ABI comparison, and preflight. The unchanged v0.1.0 Studionet address and live evidence remain historical and do not verify candidate behavior. No v0.1.1 deployment, deployed-source parity, live dispute behavior, live image review, or hosted CI for the remediation branch is claimed.
