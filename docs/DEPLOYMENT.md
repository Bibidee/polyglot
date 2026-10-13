# Polyglot deployment record

## Status

Polyglot v0.1.0 is a local, undeployed candidate. There is no current contract address, deployment transaction, source parity claim, or live lifecycle evidence.

## Freeze checklist

Before deploying to Studionet:

1. Confirm main/source commit and contract version.
2. Run `python -m pip install -r requirements.txt` in Python 3.12.
3. Run `python scripts/preflight.py`, explicit GenVM lint, and explicit schema generation.
4. Confirm exactly one deployable source exists under `contracts/` and the contract SHA-256 is recorded.
5. Push the frozen source and require GitHub Actions success on that exact commit.
6. Verify the selected unlocked CLI account and network are Studionet (chain ID 61999) before deployment.
7. Deploy only the frozen `contracts/polyglot.py` source.
8. Require finalized deployment, successful GenVM execution, `get_info()` version 0.1.0, and byte-for-byte deployed-source parity.
9. Run a fresh live task lifecycle with independent publisher and translator accounts: create funded task, accept, submit hash-bound translation, review, then settle only after the canonical state is approved or rejected.
10. Record finalized transaction links, canonical reads, child transfer result/credited value where exposed, and final source SHA.

Do not treat an accepted-but-unfinalized deployment or write as complete. Do not claim parity without retrieving exact deployed bytes. Do not claim a payout was received merely because the contract entered `payout_dispatched`.
