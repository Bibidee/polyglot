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

The v0.1.1 candidate was committed at `4f8a1ab5250a3279dfec440c3b8558cf80fa45eb`; its source SHA-256 was `ddbc3b6c69509fae227bfd59a1492887ac06e2023da74ceafe842dbbd6cf510f`. GitHub Actions run [38430058935](https://github.com/Bibidee/polyglot/actions/runs/38430058935) completed successfully for that candidate: 80 Direct Mode tests passed, with 0 failures and 0 skips, and lint, ABI, and preflight passed. This is historical evidence for v0.1.1 only, not a hosted CI result for the current v0.1.3 candidate.

## v0.1.2 remediation candidate — superseded, NOT DEPLOYED

Candidate commit `083e53487e9baa6ce413a39a456eaf21ea4e3f4f` and hosted CI run [38435878608](https://github.com/Bibidee/polyglot/actions/runs/38435878608) are historical v0.1.2 evidence only. The candidate had 99 passing Direct Mode tests; it was not deployed and has no source-parity or live-lifecycle evidence. Its single fallback-share API is superseded by v0.1.3. The v0.1.0 deployment above remains untouched and is the only live deployment.

## v0.1.3 security remediation candidate — NOT DEPLOYED

The current remediation candidate changes task creation and stored terms: the publisher pre-sets `non_submission_translator_bps` and `unresolved_translator_bps` independently, and the named translator accepts both before funding. `non_submission_translator_bps` applies only if no translation is submitted by `completion_by`; `unresolved_translator_bps` applies only if a submitted task remains unresolved at `review_deadline`. Each share is bounded to 0–10,000 basis points, immutable after creation, and the other party may refuse the terms before funding. These are explicit scenario-specific allocations, not findings of fault or proof of work.

Confidence is defined as overall confidence in the complete semantic classification. Both `approved` and `rejected` require confidence >=80. A lower-confidence apparent approval or material-defect classification is `inconclusive`, which cannot directly authorize settlement. An unresolved task may still settle at its applicable deadline using the corresponding pre-agreed share.

The review budget is 12 ordinary finalized calls plus one reserved recovery call, and up to three ordinary semantic assessments plus one reserved assessment. Operational/evidence failures consume ordinary calls but not semantic assessments. The one reserve becomes available after either ordinary-call exhaustion or three semantic assessments, requires the normal cooldown, must occur before the fixed review deadline, and is never replenished. The bounded reserve is tested after twelve provider, source-availability, and source/translation hash failures, including recovery at the final reviewable second.

`get_info()` reports the ordinary call budget (`12`), the one-call reserve, maximum total review calls (`13`), ordinary semantic assessment limit (`3`), maximum semantic assessments including reserve (`4`), and the minimum confidence (`80`). This avoids representing the ordinary limit as the total call cap.

Local candidate source snapshot: SHA-256 `13a66b47654eb8168ef0a37bb8f94fd687c29b69b546702e3c231dedd9c0850f`; Git blob `5bc9af022446427ebce23e78b6c747122958c909`; `42,674` bytes. Direct Mode: `161 passed`, `0 skipped`, `0 failed`; lint, schema, and preflight passed on this source snapshot. Hosted CI for the final pushed commit must still be checked separately. This source is not deployed and has no v0.1.3 live evidence.

The accounting invariant and child-transfer limitation remain unchanged from v0.1.2: liabilities are cleared and per-recipient intended amounts recorded before dispatch; two split transfers can have different child outcomes, and dispatch is not proof of recipient credit. Reconcile each recipient against the child receipt, `value_credited`, and balance. Do not blindly retry a failed child. Historical live evidence above is exclusively for v0.1.0 and does not certify v0.1.3.

Release status: source/testing changes are being validated on `codex/polyglot-security-remediation`. The new source has not been deployed. Any future deployment requires a signer explicitly authorized for Polyglot on Studionet chain ID `61999`; no account from another project may be assumed authorized. Do not describe v0.1.0's address as v0.1.3 or claim source parity/live evidence for this candidate. The exact frozen source hash, ABI, full test/lint/preflight results, hosted CI, and deployment authorization status are recorded after the release gates complete. See [DESIGN.md](DESIGN.md) for the full state machine and [README.md](../README.md) for the usage and limitations.
