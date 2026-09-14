# Evidence table

Review 2026-09-14. Tag `v1.0.0-rc.1` @ `040b1ec`.

## kaspa-x402

| Claim | Evidence | Verdict |
| --- | --- | --- |
| x402 v2 HTTP headers | `spec/http-profile.md`; demo `/supported` `x402Version: 2` | **live** (wire) |
| Native KAS only | `asset: "KAS"`; Luke [2099323011556106616](https://x.com/elldeeone/status/2099323011556106616) | **live** |
| `kaspa:testnet-10` syntax | demo `/supported`; schemas enum | **live** (syntax) |
| CAIP namespace registered | [namespaces#193](https://github.com/ChainAgnostic/namespaces/pull/193) open | **not registered** |
| In upstream x402 SDKs | foundation README: no Kaspa package | **wrong** if claimed |
| Mainnet ready | `docs/mainnet-readiness.md`; demo network testnet-10 | **wrong** |
| v1 RC1 exists | GitHub tag; demo health `1.0.0-rc.1` | **live** (RC, not v1) |
| Spec frozen | `spec/kaspa-x402-v1.md` Status: draft | **alpha** |
| Exact tiny payment on TN10 | report tx `b46de1e1…`; TN10 API `is_accepted: true` | **alpha** (their harness + this API check) |
| Exact replay rejection | this machine: proof:offline replay `409 invalid_transaction_state` | **live** (offline) |
| Amount locked on claim successor | `.sil` `outputs[1].value == inputValue - claimAmount` | **live** (this contract) |
| `validateOutputState` locks value | not used | **N/A** (avoided) |
| Foreign `readInputState` safe | unused; silverscript **#234** closed unmerged | **wrong** as a general Kaspa claim |
| `State[].split()` tuples safe | unused; **#249/#250** open | **wrong** on v1.0.0 |
| Compute budget from compiler | **#243** open; local 9999/10000 | **wrong** |
| 30 conf = Kaspa finality | README: deployment policy | **wrong** if claimed consensus |
| `npm test` on Windows | this machine: CRLF hash fail; 0600 fail | **broken** on default checkout |
| Offline protocol checks | 22/22 `ok: true` then fsync EPERM | **live** with a Windows write-out fail |
| Independent audit | readiness gate open | **absent** |
| Stables work without new binding | Luke + hardcoded KAS | **wrong** |

## Ishum

| Claim | Evidence | Verdict |
| --- | --- | --- |
| Speaks x402 v2 | zero `PAYMENT-*` / `x402Version` hits | **wrong** |
| Settles live KAS | watch `api.kaspa.org`, `kaspa:` URI | **live** (POS) |
| kUSD is money | `Live: false`, demo settle, no reserves | **wrong** / reserved chair |
| USDT is Kaspa money | guest IOU, freeze, demo settle | **wrong** |
| 10 conf ≈ 10 seconds | README leftover; mainnet is 10 BPS | **stale** |

## Master file

| Claim | Evidence | Verdict |
| --- | --- | --- |
| R-X402 still “TN10 alpha” on 14 Sep morning | intel-pack / THINK-BIG vs GitHub tag 13 Sep | **was stale** |
| Pin updated this pass | [662c77a](https://github.com/STP-KAS/kaspa-master-file/commit/662c77a) | **fixed** |
| k402 is Kaspa’s x402 | kccs#4 open dirty; Kali custom `kaspa-channel` | **wrong** |
| KCC-0402 adopted | PR open | **wrong** |
