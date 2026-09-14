# Battle 3 — Kaspa master file

The master file is this desk’s public pin list. It is not Kaspa core. Merged Active KIP = law. A tweet is not. Open PR ≠ a pin.

Repo: [STP-KAS/kaspa-master-file](https://github.com/STP-KAS/kaspa-master-file)  
Earlier grok pass on the freeze: [GROK-HEAVY-REVIEW.md](https://github.com/STP-KAS/kaspa-master-file/blob/main/GROK-HEAVY-REVIEW.md)  
This chapter is the **402 pin battle** from 14 Sep 2026, after Luke tagged RC.1.

---

## grok test

What the file said on the morning of 14 Sep, before this pass:

| Field | Was |
| --- | --- |
| R-X402 | “Kaspa x402 v2 binding (**TN10 alpha**). Bind this envelope.” |
| THINK-BIG kaspa-x402 row | “**TN10 alpha.** Bind this envelope.” |
| Step 3 (charge for the call) | “Bind elldeeone/kaspa-x402 (**TN10 alpha**, kaspa-x402.org)” |

What GitHub said the same morning:

| Field | Live |
| --- | --- |
| Latest tag | **`v1.0.0-rc.1`** published 13 Sep 2026 14:47 UTC |
| Release notes | First v1 RC for Testnet-10. Mainnet blocked. 18 required flows on a funded TN10 run |
| Hosted gateway | `releaseVersion: 1.0.0-rc.1` |
| Prior tag | `v0.1.0-alpha.10` (10 Aug) — that is the tag the pin was describing |

The pin was **one release behind**. Same class of error the 11 Sep grok heavy review already caught on SilverScript: intel still saying `v1-rc1` after Ori had tagged **v1.0.0**.

k402 / kccs#4 rechecked: still **open**, `mergeable_state: dirty`. Not adopted. Not x402 v2.

Compiler holes rechecked: silverscript **#234** closed unmerged, **#243** open, **#249** open, **#250** open. Unchanged.

---

## grok analyse

The master file has one job for 402: **name the envelope** and **refuse the costumes**.

Desk law (THINK-BIG) that was already correct:

- **P6.** Bind one 402 envelope. elldeeone/kaspa-x402. Steal k402’s lock/voucher. Credit both. Never a fourth envelope. Never “adopted KCC-0402.”
- Skip centralised stables for dapps.
- Dollars column **0–0**. Do not invent kUSD as a peg.
- Compiler pin SilverScript v1.0.0. Amount is not locked by `validateOutputState` unless you `require(value)`.

What was wrong was only the **status word**: “alpha” after RC.1 existed.

Updated in [662c77a](https://github.com/STP-KAS/kaspa-master-file/commit/662c77a):

- Bind **v1.0.0-rc.1**
- TN10 only, **mainnet blocked**
- Point at this grok heavy test
- k402 wording unchanged: steal the lock, not the name

---

## grok reasoning

A pin list that lags a tag teaches builders to wait for something that already shipped, or to treat RC as “still just alpha” without reading the gates.

RC.1 is **not** v1. The master file must not swing the other way and say “x402 is live on Kaspa.” Honest label:

> intended x402 v2 binding. Tag v1.0.0-rc.1. TN10 only. Mainnet blocked. Bind this envelope.

That is the same voice as: Toccata **live**, Argent **preview**, KCC-0020 **draft**, DAGKnight **research**.

The fourth-envelope kill-if is the one this desk is most likely to break by accident: naming a till, a CDN, or a `kns` HTTP 402 route as x402. Battle 2 and [05-402-is-not-x402.md](05-402-is-not-x402.md) exist so we do not.

---

## grok advice

1. Keep R-X402 on **RC.1** until a real `v1.0.0` (not rc) exists **and** the mainnet gates close. Then still do not say mainnet until independent evidence exists.
2. Do not promote k402 when it gets a new README. Recheck kccs#4 state. Open + dirty ≠ adopted.
3. Monday intel job must fail if the kaspa-x402 **latest tag** disagrees with the freeze table. That is how the compiler-tag bug comes back.
4. Do not put Ishum, kns 402, or kascade in the x402 pin row.
5. Compiler holes stay on the freeze table until they close. kaspa-x402 avoiding them is not the compiler fixing them.
