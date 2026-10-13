# Polyglot deployment and live verification

## Current Studionet deployment — v0.1.3

- Network: GenLayer Studionet, chain ID `61999`
- Contract: [`0x7B68e7E83BcBF6290c2Ef2C78663059ae0D131F8`](https://explorer-studio.genlayer.com/address/0x7B68e7E83BcBF6290c2Ef2C78663059ae0D131F8)
- Deployment transaction: [`0x8e580d0c2fd418c62bd2f0c8b5f0246b1c7b297a3348049cee2911530fea23e0`](https://explorer-studio.genlayer.com/tx/0x8e580d0c2fd418c62bd2f0c8b5f0246b1c7b297a3348049cee2911530fea23e0)
- Version: `0.1.3`
- Frozen source commit: `a2a14391c643e0344f79ec28992f837dabe9d6b2`
- Contract SHA-256: `13a66b47654eb8168ef0a37bb8f94fd687c29b69b546702e3c231dedd9c0850f`; Git blob `5bc9af022446427ebce23e78b6c747122958c909`; source length `42,674` bytes.
- Source parity: **verified byte-for-byte** through the Explorer's Contract → Code source. Its raw UTF-8 text was hashed and matched the repository SHA-256 and byte length.
- Deployment receipt: `FINALIZED`, `MAJORITY_AGREE`, leader GenVM `SUCCESS`.
- `get_info()`: `name=Polyglot`, `version=0.1.3`, `min_escrow_wei=1000000000000000`, `minimum_confidence=80`; the live call also reported the configured review budgets and transfer semantics.
- Release gate for the frozen source commit: 161 Direct Mode tests passed, 0 skipped/failed; GenVM lint, ABI/schema, and preflight passed. GitHub Actions [run 38442885261](https://github.com/Bibidee/polyglot/actions/runs/38442885261) completed successfully.

## v0.1.3 live approved-and-paid lifecycle

This fresh task was executed on the v0.1.3 deployment using two Polyglot-specific accounts: publisher `0x7AFc209aD72153f6c8Ae7B8055bfbA6E7f4d5EfC` and designated translator `0x8F41F2dA6636Eb802fa138b9558635c765fF211d`. No credentials are recorded here. The task used commit-pinned artifacts from repository commit `9e49bb1f4b6f39655690599d9e0f38a8af30ea6d`:

- Source notice: [live_source_notice.txt](https://raw.githubusercontent.com/Bibidee/polyglot/9e49bb1f4b6f39655690599d9e0f38a8af30ea6d/evidence/live_source_notice.txt), SHA-256 `0xfe82020d464226218f0dfedc347023d68e83c48a50a04325de5834f8d802c05b` (127 exact raw bytes).
- Spanish translation: [live_translation_es.txt](https://raw.githubusercontent.com/Bibidee/polyglot/9e49bb1f4b6f39655690599d9e0f38a8af30ea6d/evidence/live_translation_es.txt), SHA-256 `0x0763688d89cc6a6893f281b10737103d382b2c344683bab291accb244511bdd6` (142 exact raw bytes).
- Task: `PG-CODEX-LIVE-1791933532141`; brief required checking that the Spanish notice preserved opening hours `09:00–17:00 Monday through Friday` and closure on public holidays.
- Deposit: `0.001 GEN` (`1000000000000000` wei); task accounting was solvent while funded and returned to zero liabilities after settlement.

| Action | Transaction | Finalized result |
| --- | --- | --- |
| Create task | [`0x340d0b96f9f3e4fc3df4d9a4e2ca6c7ca017d500fd54a511858ff329cba02f6d`](https://explorer-studio.genlayer.com/tx/0x340d0b96f9f3e4fc3df4d9a4e2ca6c7ca017d500fd54a511858ff329cba02f6d) | `FINALIZED`, `MAJORITY_AGREE`, GenVM `SUCCESS`; state `awaiting_acceptance`. |
| Translator accepts | [`0x0a22250c4a8e82b748fcbc0db84fa19aa16e258a4a21f0c05c9c1b8a65c9ad47`](https://explorer-studio.genlayer.com/tx/0x0a22250c4a8e82b748fcbc0db84fa19aa16e258a4a21f0c05c9c1b8a65c9ad47) | `FINALIZED`, `MAJORITY_AGREE`, GenVM `SUCCESS`. |
| Deposit publisher credit | [`0x95bbedf5a41217a96f0b7dacf2f640790f091fd103296d6d9c8bdfad7765591e`](https://explorer-studio.genlayer.com/tx/0x95bbedf5a41217a96f0b7dacf2f640790f091fd103296d6d9c8bdfad7765591e) | `FINALIZED`, `MAJORITY_AGREE`, GenVM `SUCCESS`; `value_credited=true`. |
| Fund task | [`0x85c666dceeb2e394e7fd2bb57808d3b0e3754fc5bca213b2bf5a45a786451739`](https://explorer-studio.genlayer.com/tx/0x85c666dceeb2e394e7fd2bb57808d3b0e3754fc5bca213b2bf5a45a786451739) | `FINALIZED`, `MAJORITY_AGREE`, GenVM `SUCCESS`; state `pending`, deposited `1000000000000000` wei. |
| Submit translation | [`0xc13c93f94b8f8aaf74ffe27a55440658f7e24e9481be82b359e20c62b32c2530`](https://explorer-studio.genlayer.com/tx/0xc13c93f94b8f8aaf74ffe27a55440658f7e24e9481be82b359e20c62b32c2530) | `FINALIZED`, `MAJORITY_AGREE`, GenVM `SUCCESS`; state `submitted`. |
| Semantic review | [`0xd7ecec3b18cd8edbecfb50649f9426d049f37461f5e7b2001e27f1737184546a`](https://explorer-studio.genlayer.com/tx/0xd7ecec3b18cd8edbecfb50649f9426d049f37461f5e7b2001e27f1737184546a) | `FINALIZED`, `MAJORITY_AGREE`, GenVM `SUCCESS`, 5/5 votes revealed in one round; state `approved`, confidence `98`. |
| Settle payout | [`0x199d3cdb3e6e9fda520a8c5f1cdc05b50ea0a1511a2a0b3de4be783eaa11fd36`](https://explorer-studio.genlayer.com/tx/0x199d3cdb3e6e9fda520a8c5f1cdc05b50ea0a1511a2a0b3de4be783eaa11fd36) | `FINALIZED`, `MAJORITY_AGREE`, GenVM `SUCCESS`; state `payout_dispatched`, deposited `0`, translator dispatch `1000000000000000` wei. |
| Child transfer credit | [`0x5689fd6b15382c7bce1aee884dc5d2201a0f15d8cc765b0106e84eb5a6458379`](https://explorer-studio.genlayer.com/tx/0x5689fd6b15382c7bce1aee884dc5d2201a0f15d8cc765b0106e84eb5a6458379) | `FINALIZED`, `value_credited=true`, `1000000000000000` wei credited to the designated translator. The recipient balance read was `1001000000000000000` wei. |

The stored rationale was: “Source is English and translation is Spanish, both matching declarations. All material content is faithfully preserved: opening hours 09:00–17:00, Monday through Friday, and closure on public holidays are accurately rendered. No terms added or omitted. No discrepancy introduces or conceals any material risk.” This proves the reviewed approval and actual child credit. The parent task's terminal state remains `payout_dispatched`; do not treat that state alone as proof of payment. The separate child receipt and recipient balance provide the credit evidence.

The earlier v0.1.0 lifecycle below is historical and is not evidence for the current deployment.

## Historical v0.1.0 live approved-and-paid lifecycle

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

The v0.1.1 candidate was committed at `4f8a1ab5250a3279dfec440c3b8558cf80fa45eb`; its source SHA-256 was `ddbc3b6c69509fae227bfd59a1492887ac06e2023da74ceafe842dbbd6cf510f`. GitHub Actions run [38430058935](https://github.com/Bibidee/polyglot/actions/runs/38430058935) completed successfully for that candidate: 80 Direct Mode tests passed, with 0 failures and 0 skips, and lint, ABI, and preflight passed. This is historical evidence for v0.1.1 only, not for the current v0.1.3 release.

## Historical v0.1.2 remediation candidate — superseded, NOT DEPLOYED

Candidate commit `083e53487e9baa6ce413a39a456eaf21ea4e3f4f` and hosted CI run [38435878608](https://github.com/Bibidee/polyglot/actions/runs/38435878608) are historical v0.1.2 evidence only. The candidate had 99 passing Direct Mode tests; it was not deployed and has no source-parity or live-lifecycle evidence. Its single fallback-share API is superseded by v0.1.3. The v0.1.0 deployment above remains untouched and is superseded by the current v0.1.3 deployment.

## v0.1.3 security remediation — deployed and live-verified

The current remediation candidate changes task creation and stored terms: the publisher pre-sets `non_submission_translator_bps` and `unresolved_translator_bps` independently, and the named translator accepts both before funding. `non_submission_translator_bps` applies only if no translation is submitted by `completion_by`; `unresolved_translator_bps` applies only if a submitted task remains unresolved at `review_deadline`. Each share is bounded to 0–10,000 basis points, immutable after creation, and the other party may refuse the terms before funding. These are explicit scenario-specific allocations, not findings of fault or proof of work.

Confidence is defined as overall confidence in the complete semantic classification. Both `approved` and `rejected` require confidence >=80. A lower-confidence apparent approval or material-defect classification is `inconclusive`, which cannot directly authorize settlement. An unresolved task may still settle at its applicable deadline using the corresponding pre-agreed share.

The review budget is 12 ordinary finalized calls plus one reserved recovery call, and up to three ordinary semantic assessments plus one reserved assessment. Operational/evidence failures consume ordinary calls but not semantic assessments. The one reserve becomes available after either ordinary-call exhaustion or three semantic assessments, requires the normal cooldown, must occur before the fixed review deadline, and is never replenished. The bounded reserve is tested after twelve provider, source-availability, and source/translation hash failures, including recovery at the final reviewable second.

`get_info()` reports the ordinary call budget (`12`), the one-call reserve, maximum total review calls (`13`), ordinary semantic assessment limit (`3`), maximum semantic assessments including reserve (`4`), and the minimum confidence (`80`). This avoids representing the ordinary limit as the total call cap.

Frozen v0.1.3 source: SHA-256 `13a66b47654eb8168ef0a37bb8f94fd687c29b69b546702e3c231dedd9c0850f`; Git blob `5bc9af022446427ebce23e78b6c747122958c909`; `42,674` bytes. Direct Mode: `161 passed`, `0 skipped`, `0 failed`; lint, schema, and preflight passed. The initial release-gate run [38442040338](https://github.com/Bibidee/polyglot/actions/runs/38442040338) passed for source commit `1c6ddc779f009996ce11e3bcc7e1a1509f283b3a`; the frozen release commit's successful CI is [run 38442885261](https://github.com/Bibidee/polyglot/actions/runs/38442885261). Deployment finality and byte-for-byte source parity are recorded at the top of this document; the live lifecycle is recorded above.

The accounting invariant and child-transfer limitation remain unchanged from v0.1.2: liabilities are cleared and per-recipient intended amounts recorded before dispatch; two split transfers can have different child outcomes, and dispatch is not proof of recipient credit. Reconcile each recipient against the child receipt, `value_credited`, and balance. Do not blindly retry a failed child. The v0.1.0 evidence below is historical and does not certify v0.1.3; current v0.1.3 deployment and lifecycle evidence is recorded above.

Release status: v0.1.3 is deployed on Studionet and the source parity, release gate, and live approval/payout lifecycle are verified. The v0.1.0 deployment and its separate failed payable-call evidence remain historical; do not present that address as the current release. See [DESIGN.md](DESIGN.md) for the full state machine and [README.md](../README.md) for usage and limitations.
