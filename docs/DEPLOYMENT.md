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

## v0.1.1 hardening candidate (not deployed)

The current branch contains an unreleased v0.1.1 source change; it does not alter the v0.1.0 address or its state. Candidate source SHA-256 is `ddbc3b6c69509fae227bfd59a1492887ac06e2023da74ceafe842dbbd6cf510f` (36,606 bytes at the latest local preflight). Direct Mode has 80 passed, 0 skipped, 0 failed; GenVM lint, ABI/schema comparison, and preflight pass locally. No v0.1.1 GitHub Actions run, deployment, source parity, or live lifecycle has been verified.

Task creation is nonpayable and records validated terms in `awaiting_funding`. Payable `deposit_credit()` records GEN only as available credit for the actual sender. A separate nonpayable `fund_task(task_id)` allocates the exact task amount from the publisher's own credit; excess remains withdrawable credit. Ordinary task-ID, authorization, underfunding, and duplicate-allocation errors occur on the nonpayable call, so they do not attach value. This cannot guarantee accounting if a runtime failure prevents the minimal payable method from persisting its state.

Permanent evidence-integrity failures become `disputed`, not synthetic semantic rejection. After submitted work, an expired review deadline also moves unresolved tasks to `disputed`; the publisher-only timeout refund is retained only before submission. Disputed funds can be split only when both participants agree to the exact same translator share. There is no unilateral adjudicator or dispute timeout; if the parties do not agree, funds remain locked. This is a known liveness/economic limitation, not a completed recovery path.

The candidate changes the public API to create, deposit credit, then allocate. Do not use the current v0.1.0 deployment as if it supported this API. A new deployment requires an explicit release authorization and signer; no candidate deployment is claimed here.

## Baseline audit and finding status

The audited starting point was `0fcf9697fbd976832cd3596996184ca1926cbb4b`. Its Direct Mode suite was independently reproduced at 63 passed, 0 failed. Source and recorded live evidence confirmed the four reported findings: (H-01) v0.1.0's payable task creation could leave credited GEN without task accounting after validation failure; (H-02) artifact-integrity failure was converted to semantic rejection and could make publisher refund available; (H-03) `review()` was callable by unrelated wallets and charged attempts before nondeterministic success; (H-04) terminal settlement recorded dispatch, not guaranteed recipient credit.

Candidate disposition: H-01 is mitigated for ordinary task-input mistakes by nonpayable validation plus separate sender-credit deposit/allocation; runtime failure in `deposit_credit()` remains a platform/custody limitation. H-02 no longer turns artifact integrity failure into semantic rejection; it freezes funds for bilateral settlement. H-03 review is limited to task participants, uses cooldown, and only valid semantic results consume the three-review assessment count; retries remain bounded by the seven-day review period, but can be repeatedly attempted at five-minute intervals by a participant and are not free of transaction costs. H-04 is not solvable inside this contract without a supported child-transfer callback/reconciliation primitive; dispatch is terminal and no blind retry exists. The exact evidence and code/test limits are documented in [DESIGN.md](DESIGN.md).

## Release checks and limitations

The v0.1.0 source is frozen at the commit listed above; documentation may advance independently. The v0.1.1 candidate has not been deployed, so v0.1.0 remains the only verified address. Neither version guarantees consensus finalization, legal correctness, external host availability, DNS/redirect safety, or recovery if a runtime failure prevents a payable call from persisting accounting. Candidate disputes can remain locked without bilateral agreement. `payout_dispatched` alone is not proof of recipient credit; only the child receipt and recipient balance recorded above establish the historical v0.1.0 payout.
