# Windows repro — paste this as an issue

**Filed:** https://github.com/elldeeone/kaspa-x402/issues/11
**PR:** https://github.com/elldeeone/kaspa-x402/pull/12

Open (if filing again): https://github.com/elldeeone/kaspa-x402/issues/new

## Title (copy)

```
Windows: npm test fails on v1.0.0-rc.1 (CRLF .sil hash, 0o600, fsync)
```

## Body (copy)

```
Agent pass of `v1.0.0-rc.1` (`040b1ec`) on Windows (Node 24, `core.autocrlf=true`).

Full writeup: https://github.com/STP-KAS/grok-heavy-test

CI is `ubuntu-latest` only, so this never shows.

## 1. Covenant launch-identity hash (default `npm test` fail)

No `.gitattributes`. `contracts/kaspa-x402-escrow-v4.sil` checks out CRLF.

| | sha256 of `.sil` |
| --- | --- |
| CRLF on disk | `ab88ec5da53d1e716bb1f1322182d48bbb2c9fed9b0078f9d718a022282f48c6` |
| LF (pinned `ESCROW_V4_SOURCE_SHA256`) | `065dff5d0d02f3a09f56bab977a33d4e047f2ccec64c0d066a318d342797fcb0` |

Fails:
- `packages/covenant/test/covenant.test.ts` → reproduces current silverc genesis and successor bytecode
- same file → binds the launch identity to the checked source and compiled base

After deleting `\r` from the working tree (not committed), those 27 tests passed. Bytecode pins were fine; only the source-file hash moved.

Fix: `*.sil text eol=lf` in `.gitattributes`, and hash LF-normalized bytes.

## 2. Unix 0600 asserted on Windows

`scripts/proof-output-security.test.mjs` expects `mode & 0o777 === 0o600` (384). This machine got 438 (`0o666`). Secret-scrub checks are fine; the mode assert is POSIX-only.

## 3. `proof:offline` then `fsync`

22/22 named checks `ok: true` (exact replay `409 invalid_transaction_state`, batch genesis/claim/top-up/refund construction, fixture pin `3ed9733`). Summary then:

```
"ok": false, "error": "EPERM: operation not permitted, fsync"
```

Protocol path passed. Report write-out did not.

Happy to retest a branch.
```

## Why this matters

Launch identity is supposed to bind the **checked source** to the compiled bytecode. If Windows Git changes the bytes, two honest checkouts of the same tag produce two hashes. That is exactly the class of drift the pin exists to catch — except here the drift is the line ending, not the contract.

An agent asked to “run the tests” on Windows cannot complete the advertised path without knowing the LF trick. Luke asked agents to go through it. This is what one of them hit first.
