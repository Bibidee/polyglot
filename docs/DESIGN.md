# Polyglot design

## Purpose

Polyglot escrows a publisher-funded translation reward while validators independently check whether a hash-committed translation preserves the material meaning of a hash-committed source notice. It is designed for translations where omission or alteration of a right, duty, prohibition, deadline, amount, condition, exception, or scope is consequential.

The contract does not translate documents, certify lawfulness, prove authorship, or establish real-world identities. It adjudicates bounded semantic parity evidence supplied by participants.

## State machine

| State | Meaning | Legal next states |
|---|---|---|
| `pending` | Publisher funded; named translator has not accepted. | `accepted`, `refund_dispatched` |
| `accepted` | Named translator accepted but has not submitted. | `submitted`, `refund_dispatched` |
| `submitted` | Translation and optional image commitments are stored. | `approved`, `rejected`, `inconclusive`, `retryable` |
| `retryable` | Source/model/network execution failed safely. | `approved`, `rejected`, `inconclusive`, `retryable`, `refund_dispatched` |
| `inconclusive` | Valid review could not establish parity or a material mismatch. | `approved`, `rejected`, `inconclusive`, `refund_dispatched` |
| `approved` | Full safe semantic tuple reached consensus. | `payout_dispatched` |
| `rejected` | A definitive mismatch or material risk reached consensus. | `refund_dispatched` |
| `payout_dispatched` | Reward transfer was emitted to the stored translator. | Terminal |
| `refund_dispatched` | Escrow refund was emitted to the stored publisher. | Terminal |

The translator cannot change the committed source or replace a translation after submission. The publisher can cancel only in `pending`. A bounded attempt count prevents repeated review submissions from becoming unbounded state churn. Public timeouts provide a publisher refund when acceptance, translation, or review does not resolve in time.

## Consensus boundary

Each leader/validator independently retrieves the source, translation, and optional images, verifies exact raw-byte hashes, decodes text, and derives a bounded semantic result. Storage is copied to a local snapshot before entering nondeterministic callbacks; callbacks do not read or mutate persistent storage.

Approval is derived from all required conditions; the model never supplies a recipient, amount, status transition, or payout fraction. Equivalence requires agreement on `kind` and derived `outcome`. For `approved`, both validated outputs necessarily satisfy the complete positive tuple. Rationales and exact confidence values are not compared because they do not change authorization. Different valid reasons for `rejected` or `inconclusive` can agree safely because neither outcome pays the translator. An approval/non-approval disagreement is rejected by the validator.

Provider and web failures are non-authorizing and become retryable when safely classified. A mismatch, invalid encoding, empty/oversized artifact, or unsupported image cannot approve. If a model response is malformed, it becomes retryable rather than being loosely parsed into approval.

## Escrow

The task creator funds the reward with a payable write. `gl.message.value` is authoritative; caller-supplied amount fields do not exist. The agreed `reward_wei` term is recorded separately from `deposited`, the actual remaining liability. Settlement reads only the ledger, clears it, records the terminal state and amount, then emits a GEN transfer through one helper. A task cannot dispatch a second payout/refund because its state is terminal and its ledger is zero.

The contract can record only that it dispatched a child transfer. It cannot equate dispatch with recipient credit. Operators must inspect the downstream finalized transfer receipt and credited value when available.

## Visual evidence

At task creation, the publisher may commit a source scan/image. At translation submission, the named translator may commit a translated scan/image. Each optional image has an HTTPS URL and SHA-256. The exact raw image bytes are size-limited, hash-checked, signature-checked as PNG/JPEG/WebP, then sent to `exec_prompt(images=[...])`. Images supplement the text comparison; ambiguous or contradictory visual evidence cannot satisfy approval.

## Trust assumptions and limits

- Source notice authenticity and legal authority are outside the contract's proof.
- URL filtering is not DNS pinning and cannot guarantee redirect safety.
- LLM/vision judgments can be wrong, vary across validators, or fail to finalize.
- Hashes bind bytes, not truth or authorship.
- Honest fulfillment depends on someone submitting acceptance, settlement, or timeout transactions.
- Failed external transfer children may not be recoverable.
