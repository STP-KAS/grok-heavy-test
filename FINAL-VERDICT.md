# Final verdict

**One GitHub. This page.** Date: 2026-09-14. Subject: [elldeeone/kaspa-x402](https://github.com/elldeeone/kaspa-x402) tag **`v1.0.0-rc.1`** (`040b1ec`). Machine: Windows, Node 24, `core.autocrlf=true`.

Not Kaspa core. Not an audit. Not a v1 rubber stamp.

---

## Verdict (one paragraph)

**kaspa-x402 is a real x402 v2 binding for native KAS.** Bind this envelope. Do not invent a fourth. **It is not v1, not mainnet, not in the upstream x402 registry, not a registered CAIP namespace.** Default Windows `npm test` was broken on clone. That is a real break of “clone it and run the tests.” It is not a break of the covenant math. Both Windows breaks are now reproduced, patched, filed, and re-tested on this machine.

**Ishum is a coffee till, not x402.** Zero `PAYMENT-*` headers. Quotes EUR/USD. Settles native KAS. Do not weld the names.

---

## Both problems — solved on this pass

| # | Break | Status |
| --- | --- | --- |
| 1 | Default Windows `npm test` fails: `.sil` checks out CRLF, launch-identity SHA-256 is pinned to LF (`ab88ec5d…` vs `065dff5d…`) | **Reproduced. Patched. 27/27 covenant tests pass.** |
| 2 | Unix `0o600` asserted (expect 384, Windows 438) **and** `proof:offline` dies on `fsync` EPERM after 22/22 protocol checks | **Reproduced. Patched. `npm test` green. `proof:offline` 24/24 `ok: true`.** |

Filed: [elldeeone/kaspa-x402#11](https://github.com/elldeeone/kaspa-x402/issues/11)
PR: [elldeeone/kaspa-x402#12](https://github.com/elldeeone/kaspa-x402/pull/12)
Patch: [patches/windows-clone-and-test.patch](patches/windows-clone-and-test.patch)

The patch:

- `*.sil text eol=lf` in `.gitattributes`
- hash LF-normalized source bytes (`sha256SilSource`)
- skip Unix `0o600` / `chmod` on `win32`
- treat `fsync` `EPERM` / `EISDIR` / `EINVAL` as non-fatal so write-out cannot fail closed after protocol checks pass

A `windows-latest` CI job is the right follow-up. This token cannot push workflow files (`workflow` scope missing). Luke can add that job in-tree.

---

## What we actually ran (this session)

```
git clone --branch v1.0.0-rc.1 https://github.com/elldeeone/kaspa-x402.git
# HEAD 040b1ec, core.autocrlf=true, .sil i/lf w/crlf, no .gitattributes
npm ci
npm --workspace @kaspa-x402/covenant test   # FAIL: ab88ec5d vs 065dff5d
# apply patch
npm --workspace @kaspa-x402/covenant test   # 27 pass
npm run build && npm test                   # all workspaces + 18 script tests pass
npm run proof:offline                       # 24/24 ok: true
```

Live, no wallet:

| Probe | Result |
| --- | --- |
| `https://demo.kaspa-x402.org/health` | `ok`, `releaseVersion: 1.0.0-rc.1`, `hostedExactSettlementEnabled: true` |
| `https://demo.kaspa-x402.org/supported` | `x402Version: 2`, `exact` + `batch-settlement`, **`kaspa:testnet-10` only**, asset `KAS` |
| TN10 API `b46de1e1…` | `is_accepted: true`, `version: 0` |

Did **not** run a funded live harness. Did **not** pay the Worker.

---

## Steps to take

### Luke (owner of the envelope)

1. Read this page. Then [docs/00-for-luke.md](docs/00-for-luke.md) if you want the long form.
2. Accept or comment on [issue #11](https://github.com/elldeeone/kaspa-x402/issues/11).
3. Merge [PR #12](https://github.com/elldeeone/kaspa-x402/pull/12), or land an equivalent: `.gitattributes`, LF-normalized hash, skip Unix modes on `win32`, don’t let directory `fsync` EPERM fail `proof:offline`.
4. Add a `windows-latest` CI job (`npm ci && npm test && npm run proof:offline`). This fork could not push `.github/workflows`.
5. Align `pragma silverscript ^0.1.0` with the v1.0.0 `3ed9733` compile. Fixture bytecode pin is the real pin.
6. Do not drop the `rc` until the gates you already wrote in `docs/mainnet-readiness.md` are closed. Do not take “Kaspa is in x402” until upstream lists you or rejects you in writing.

### This desk

1. Bind **this** envelope on TN10. One local URL → 402 → pay → 200 + txid. No fourth dialect. No kUSD as this `asset`.
2. Keep Ishum a till. Default `EnableKUSD` / `EnableUSDT` **off**. Fix the 1 BPS leftover in the Ishum README (mainnet is 10 BPS).
3. Keep the master-file pin on **v1.0.0-rc.1**, TN10 only, mainnet blocked. Monday intel must fail if the latest tag disagrees with the freeze table.
4. Steal k402’s lock if a channel is needed. Credit Kali. Never “adopted KCC-0402.” Never “k402 is x402.”

### Anyone else

HTTP 402 ≠ x402. Native KAS ≠ USDC. `accepted` ≠ irreversible. A till that quotes EUR is allowed. Calling it x402 is not.

---

## What to do next

| When | Do |
| --- | --- |
| **Today** | Send Luke this page + [issue #11](https://github.com/elldeeone/kaspa-x402/issues/11) + [PR #12](https://github.com/elldeeone/kaspa-x402/pull/12). DM copy is in [SEND-THIS.md](SEND-THIS.md). |
| **This week** | Bind exact `standard-native` on TN10 in our own tree. Do not pre-sign batch ceiling **T** beyond delivered work. |
| **Before any “v1” sentence** | Independent covenant + replay audit (exact **and** batch, including the claim/refund race). Fresh funded TN10 run **and** a paid hosted-Worker canary. Durable store. Confirmation policy written down. |
| **Before mainnet KAS** | Every gate in `docs/mainnet-readiness.md`. `allowMainnet` is a boolean, not a gate. CAIP [namespaces#193](https://github.com/ChainAgnostic/namespaces/pull/193) merged **or** every public sentence says “proposed identifier, unregistered.” |
| **Never** | `allowMainnet: true` early. Foreign `readInputState` / `State[].split()` in the escrow. USDC / USDT / “kaspa dollar” as this `asset`. A fourth 402 envelope. Discord flattening `accepted` into cash. |

---

## What held (keep)

- It **is** x402 v2: `PAYMENT-REQUIRED` / `PAYMENT-SIGNATURE` / `PAYMENT-RESPONSE`, `x402Version: 2`.
- Native **KAS** only. Hosted `/supported` is `kaspa:testnet-10` only.
- Claim successor **locks value** (`outputs[1].value == inputValue - claimAmount`).
- No `readInputState`, no `State[].split()`. Avoided live compiler holes. Did not fix them in silverc — don’t need to.
- Exact replay `409 invalid_transaction_state` in offline proof (this machine).
- Facilitator optional. Direct verify is the point.
- Mainnet gates written down. They did not declare victory.

## What is still not v1 (agree with Luke)

- CAIP [namespaces#193](https://github.com/ChainAgnostic/namespaces/pull/193) **open**.
- Not in upstream x402 SDKs.
- `spec/kaspa-x402-v1.md` still **draft** next to a `v1.0.0-rc.1` tag.
- Batch timeout is a **claim vs refund race**.
- Replay lives in the store. In-memory is a fixture.
- One RPC is not Byzantine-safe. The Worker still trusts one TN10 source.
- `accepted` is not irreversible.
- 500 sompi vouchers still lock ~10M sompi-class capital. Say the lockup out loud.
- Compute budget still guessed (silverscript `#243`).

---

## Objects, honest names

| Object | Honest |
| --- | --- |
| **elldeeone/kaspa-x402** | The v2 **envelope**. Bind this. TN10 RC. Not v1. Not mainnet. |
| Kali k402 / kccs#4 | A **lock**. Steal it. Not x402 v2. Not adopted KCC-0402 (PR open, dirty). |
| Ishum | A **till**. Quotes EUR/USD. Settles KAS. Zero `PAYMENT-*`. |
| KASPACOM/x402-KAS | Superseded TN12 experiment. |
| USDC-on-Base “Kaspa x402” routers | Coinbase-culture x402 + a KAS IOU. Not native KAS in the shared standard. |

---

## Links

| | |
| --- | --- |
| **This page (send this)** | https://github.com/STP-KAS/grok-heavy-test/blob/main/FINAL-VERDICT.md |
| This repo | https://github.com/STP-KAS/grok-heavy-test |
| For Luke (long form) | [docs/00-for-luke.md](docs/00-for-luke.md) |
| Windows issue | https://github.com/elldeeone/kaspa-x402/issues/11 |
| Windows PR | https://github.com/elldeeone/kaspa-x402/pull/12 |
| Subject | https://github.com/elldeeone/kaspa-x402 |
| Luke’s ask | https://x.com/elldeeone/status/2099316438704312512 |
| Desk mix (optional) | https://github.com/STP-KAS/delusional-stp-grok-mix |
