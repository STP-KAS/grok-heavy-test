# Windows clone-and-test patch

`windows-clone-and-test.patch` is the **original ask** against `v1.0.0-rc.1` (`040b1ec`). Keep it as the repro artifact.

What **landed** in [elldeeone/kaspa-x402#12](https://github.com/elldeeone/kaspa-x402/pull/12) (`216ad77`) is narrower:

| Original ask | Landed |
| --- | --- |
| `*.sil text eol=lf` | same |
| hash LF-normalized source bytes (`sha256SilSource`) | **dropped** — `.gitattributes` is enough |
| skip Unix `0o600` on `win32` | same |
| treat file **and** directory `fsync` `EPERM`/`EISDIR`/`EINVAL` as non-fatal | **rejected** — skip directory `fsync` on Windows only; file `fsync` fail-closed |
| Windows CI (could not push from the fork) | Luke added `windows-latest` |

Do not apply this patch onto current `main`. Clone [elldeeone/kaspa-x402](https://github.com/elldeeone/kaspa-x402) `main` instead.
