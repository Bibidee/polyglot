# Polyglot deployment and live verification

## Current Studionet deployment

- Network: GenLayer Studionet, chain ID `61999`
- Contract: [`0x1E0239EBd297b29339Ec4cA767388C79EACc9078`](https://explorer-studio.genlayer.com/address/0x1E0239EBd297b29339Ec4cA767388C79EACc9078)
- Deployment transaction: [`0xe1762be0c9fed246e0fef5644112f8ad9ed0e03d727c57225ce40f5680f81ffb`](https://explorer-studio.genlayer.com/tx/0xe1762be0c9fed246e0fef5644112f8ad9ed0e03d727c57225ce40f5680f81ffb)
- Version: `0.1.0`
- Frozen source commit: `9e49bb1f4b6f39655690599d9e0f38a8af30ea6d`
- Contract SHA-256: `498762a5c8b3426f21eb8499ee793e97d77d81a1b437430efb5378f85c3ba1d9`
- Source parity: **verified byte-for-byte** via GenLayerJS `getContractCode()` backed by `gen_getContractCode`; both local and deployed source were 28,671 bytes with the same SHA-256.
- Deployment receipt: `FINALIZED`, `MAJORITY_AGREE`, leader GenVM `SUCCESS`.
- `get_info()`: `name=Polyglot`, `version=0.1.0`, `min_escrow_wei=1000000000000000`, `minimum_confidence=80`, `max_review_attempts=3`, `retry_cooldown_seconds=300`, `review_window_seconds=604800`, `transfer_semantics=dispatch_is_not_child_transfer_credit`.
- Release gate: 63 Direct Mode tests passed, 0 skipped/failed; GenVM lint, ABI/schema, and preflight passed.
- GitHub Actions for the frozen source commit: [run 38415012174](https://github.com/Bibidee/polyglot/actions/runs/38415012174), **success**.

## Live approved-and-paid lifecycle

Task `PG-LIVE-20261013-02` used distinct publisher and translator accounts. The publisher deposited the minimum reward of `0.001 GEN` (`1000000000000000` wei). The accounts below are public addresses; no signing secrets are included.

- Publisher: `0x794678ad7e8b6c87dab33303a3a512c821e6de9a`
- Designated translator/consumer: `0x2cd419603eBa593074653930Ddc4073d4FD8fc60`
- Source notice: [commit-pinned raw artifact](https://raw.githubusercontent.com/Bibidee/polyglot/9e49bb1f4b6f39655690599d9e0f38a8af30ea6d/evidence/live_source_notice.txt), SHA-256 `0xfe82020d464226218f0dfedc347023d68e83c48a50a04325de5834f8d802c05b` (127 exact raw bytes).
- Spanish translation: [commit-pinned raw artifact](https://raw.githubusercontent.com/Bibidee/polyglot/9e49bb1f4b6f39655690599d9e0f38a8af30ea6d/evidence/live_translation_es.txt), SHA-256 `0x0763688d89cc6a6893f281b10737103d382b2c344683bab291accb244511bdd6` (142 exact raw bytes).

| Action | Transaction | Finalized result |
| --- | --- | --- |
| Create funded task | [`0x84400b910d2a779bf5b4a3ce897f2dc56a9b6db5d37fdc0e1dbcb0513f876d35`](https://explorer-studio.genlayer.com/tx/0x84400b910d2a779bf5b4a3ce897f2dc56a9b6db5d37fdc0e1dbcb0513f876d35) | `FINALIZED`, `MAJORITY_AGREE`, GenVM `SUCCESS`; canonical task state `pending`, deposited `1000000000000000` wei. |
| Translator accepts | [`0x8360ddd06c023e9eaecf50d9c224a2f4119d112c98bae9803aa37b8c8afe74b0`](https://explorer-studio.genlayer.com/tx/0x8360ddd06c023e9eaecf50d9c224a2f4119d112c98bae9803aa37b8c8afe74b0) | `FINALIZED`, `MAJORITY_AGREE`, GenVM `SUCCESS`. |
| Submit committed Spanish translation | [`0xd26d933dce568e9dd22e3dd7a5073f96f900f74b03b66fc9935878d10784e110`](https://explorer-studio.genlayer.com/tx/0xd26d933dce568e9dd22e3dd7a5073f96f900f74b03b66fc9935878d10784e110) | `FINALIZED`, `MAJORITY_AGREE`, GenVM `SUCCESS`; canonical state `submitted`. |
| Semantic review | [`0xdb96098b7962aab7271610e139ac2e0e1ce9b9cd962195aa99bfbafdc1e74198`](https://explorer-studio.genlayer.com/tx/0xdb96098b7962aab7271610e139ac2e0e1ce9b9cd962195aa99bfbafdc1e74198) | `FINALIZED`, `MAJORITY_AGREE`, GenVM `SUCCESS`; canonical state `approved`, confidence `97`. |
| Settle to translator | [`0xcd2c27d989dbd26921aff3fb3c8f2823129afeb06e090d5cff49c2234fbda6ba`](https://explorer-studio.genlayer.com/tx/0xcd2c27d989dbd26921aff3fb3c8f2823129afeb06e090d5cff49c2234fbda6ba) | `FINALIZED`, `MAJORITY_AGREE`, GenVM `SUCCESS`; canonical state `payout_dispatched`, deposited `0`, dispatched amount `1000000000000000` wei. |
| Child transfer credit | [`0x6b9f459b54cea28e531458682ef680dd8ed768c59c84ebffcd2169d6b47d8dc7`](https://explorer-studio.genlayer.com/tx/0x6b9f459b54cea28e531458682ef680dd8ed768c59c84ebffcd2169d6b47d8dc7) | `FINALIZED`, `value_credited=true`, value `1000000000000000` wei to the designated translator. Translator balance read after settlement: `449121900000000000000` wei (`449.1219 GEN`). |

The stored review rationale was: “Source is English and translation is Spanish. Meaning is preserved: library open 09:00–17:00 Monday through Friday, closed on public holidays. No material additions or omissions detected; no rights/duties issues beyond the notice content.”

## Failed initial creation attempt and stranded value

The first attempted task ID was `PG-LIVE-20261013-01`, using display names `English` and `Spanish` instead of the language tags accepted by this version (`en` and `es`). Transaction [`0x6eb61fa7e63a7995184d41b27228434636f3bb3fe521750eb3f9384cb7634788`](https://explorer-studio.genlayer.com/tx/0x6eb61fa7e63a7995184d41b27228434636f3bb3fe521750eb3f9384cb7634788) finalized with `MAJORITY_AGREE` but leader GenVM `ERROR`, `[EXPECTED] Invalid language tag`. A canonical `get_task()` read confirmed no task was created. The receipt reports `value_credited=true`; the contract balance is consequently `0.001 GEN` above its task-accounted escrow. Polyglot v0.1.0 has no recovery method for this unassociated value, so that amount is stranded in this deployment. The successful lifecycle above used a separate minimum deposit; this failed attempt is retained here as evidence and is not represented as a contract success.

This also illustrates an operational rule for v0.1.0: supply normalized language tags such as `en` and `es`, not display names. Do not send value-bearing calls until all arguments have been checked against the deployed schema and language-tag rules.

## Historical v0.1.1 candidate and hosted CI

The v0.1.1 candidate was committed at `4f8a1ab5250a3279dfec440c3b8558cf80fa45eb`; its source SHA-256 was `ddbc3b6c69509fae227bfd59a1492887ac06e2023da74ceafe842dbbd6cf510f`. GitHub Actions run [38430058935](https://github.com/Bibidee/polyglot/actions/runs/38430058935) completed successfully for that candidate: 80 Direct Mode tests passed, with 0 failures and 0 skips, and lint, ABI, and preflight passed. This is historical evidence for v0.1.1 only, not a hosted CI result for the current v0.1.2 source.

## v0.1.2 remediation candidate — NOT DEPLOYED

This branch contains a material v0.1.2 source/ABI change. The v0.1.0 deployment above remains untouched and is the only deployed version. The v0.1.2 source is not yet deployed, has no deployed-source parity proof, and has no live lifecycle evidence. Do not call the historical v0.1.0 address as if it supported v0.1.2 methods or economics.

Current local release-gate snapshot (before the documentation-only closeout commit):

- Branch: `codex/polyglot-security-remediation`
- Contract SHA-256: `9b439a833da0679fe45f810bcb4e626c6435f600159a3c3e7ba87e79f539682c`
- Contract source bytes: `40,211`
- Git blob: `d138116dac592d0fa5b790d265ede930f029d0ba`
- Direct Mode tests: `99 passed`, `0 skipped`, `0 failed`
- GenVM lint: passed (`3` checks); ABI/schema generation and exact preflight schema match: passed; preflight: passed; deployable contract sources: `1`.
- GitHub Actions run [38435878608](https://github.com/Bibidee/polyglot/actions/runs/38435878608) for candidate commit `083e53487e9baa6ce413a39a456eaf21ea4e3f4f`: **completed successfully**.
- The CI result verifies this exact code candidate only. It does not establish deployment, source parity, or live v0.1.2 transaction evidence.

The v0.1.2 task API requires the publisher to specify `fallback_translator_bps` at nonpayable creation. The named translator accepts that share and the task terms before the publisher can fund. Then the publisher uses `deposit_credit()` and `fund_task()`. Unsubmitted tasks reaching `completion_by`, and unresolved submitted tasks reaching `review_deadline`, settle through that already accepted split. Both parties can still choose a different early split by mutual agreement. Evidence commitments remain immutable; 404/408/410/425/429, redirects, 5xx, network errors and model/provider failures are retryable. Hash/integrity issues enter `disputed` but remain reviewable with the same commitments during the recovery window. Finalized review calls are capped at 12, semantic assessments at 3, cooldown grows from 5 minutes to at most 6 hours, and both completion/review deadlines have deterministic settlement paths.

For any split, the contract records the exact intended publisher and translator amounts before dispatch and clears escrow liabilities first. Child-transfer credit is not confirmed by `*_dispatched`: per the current [GenLayer value-transfer documentation](https://docs.genlayer.com/developers/intelligent-contracts/features/value-transfers), value leaves into the child message and is credited only on child activation; a failed child is not automatically refunded. The supported response is reconciliation against parent and child receipts, `value_credited`, and recipient balance. This contract has no child callback/retry path and must not blindly resend a failed amount.

### Economic limitation

The fallback share is a pre-agreed allocation of unresolved evidence/timeout risk, not proof of which participant caused a problem or how much work occurred. The publisher proposes 0–10,000 basis points and the translator may refuse before funding. If accepted, the specified share is applied at the stated deadline. It avoids indefinite escrow lockup and avoids a hard-coded arbitrary percentage, but parties must knowingly choose fair terms. There is no on-chain fault oracle.

### Baseline findings and remediation status

The audited starting point was `0fcf9697fbd976832cd3596996184ca1926cbb4b`; its Direct Mode baseline was 63 passing tests. H-01 (value attached to task-creation validation failures) is mitigated for ordinary task-input validation through staged nonpayable creation and sender-owned deposit credit; failure inside the minimal payable deposit remains a platform-level accounting risk. H-02/H-01 liveness (a single evidence issue could freeze escrow) is addressed by retryable handling for availability failures, re-review of immutable commitments after evidence-integrity issues, and the accepted deadline split. M-02 (refund authorization under outcome-only equivalence) is addressed by requiring validators to agree on every bounded semantic classification field for a `rejected` result. M-03 now directly tests attached value rejection, staged acceptance/funding, evidence recovery, review-call exhaustion, fallback boundaries, and per-beneficiary split amounts. H-04 is mitigated in observability by recording intended amount per recipient; it is not recoverable on-chain after a failed child because no supported callback/re-credit mechanism is used.

## Release and deployment status

The v0.1.2 source must pass local release gates and a new GitHub Actions run on its exact commit before deployment. Any deployment requires an explicitly authorized and unlocked signer for Polyglot on Studionet chain ID `61999`; no other project's account may be assumed authorized. No v0.1.2 deployment, address, transaction, source parity, or live tests are claimed here. Existing v0.1.0 historical deployment and lifecycle evidence above remain preserved. See [DESIGN.md](DESIGN.md) for the complete state machine and [README.md](../README.md) for usage and limitations.
