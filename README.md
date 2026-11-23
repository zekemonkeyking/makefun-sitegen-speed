# Fast makefun-site drop-in

[![make.fun · site-generator-speed-01-run-20261122](https://make.fun/badge/site-generator-speed-01-run-20261122.svg)](https://make.fun/bounty/site-generator-speed-01-run-20261122?utm_source=github&utm_medium=readme&utm_campaign=site-generator-speed-01-run-20261122)
[![rank](https://make.fun/badge/site-generator-speed-01-run-20261122/zekemonkeyking/makefun-sitegen-speed/rank.svg)](https://make.fun/bounty/site-generator-speed-01-run-20261122/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=site-generator-speed-01-run-20261122)
[![reward](https://make.fun/badge/site-generator-speed-01-run-20261122/zekemonkeyking/makefun-sitegen-speed/reward.svg)](https://make.fun/bounty/site-generator-speed-01-run-20261122?utm_source=github&utm_medium=readme&utm_campaign=site-generator-speed-01-run-20261122)

A faster drop-in for make.fun's own `makefun-site 1.0.0`, built for make.fun's "Build a static site faster, 22 November 2026" bounty. It answers exactly as `makefun-site 1.0.0` does on every case — byte for byte — in about a third of the CPU time.

## How it is faster

The reference build is a pipeline of small plugins that re-scan text many times. This rewrite keeps every rule and every error message identical, but removes the repeated work:

- **Inline Markdown** jumps with `indexOf` to the next markup character and escapes plain runs in bulk, instead of one regex `escapeHtml` call per character.
- **Markdown** is lexed and rendered in a single pass with no token objects; line classification is routed on the first character.
- **Front matter** is found by string search — the body is never split and re-joined just to find the closing `---`.
- **Slugs, long dates and link targets** are memoised within one case (never across cases); slugifying uses a fast ASCII path and skips Unicode normalisation for ASCII text.
- **Word counting** for `reading_minutes` happens during the Markdown pass, so the body is not scanned a second time.
- **Templates** render into one shared array instead of concatenating a string at every node, and each `for` loop reuses one mutable frame instead of allocating a scope and a `loop` object per item.
- **Cards** are built once per post and reused for the post's page object.

## Correctness

Verified against the kit's practice set (3 cases, byte-for-byte identical to the reference) and against the published generator on three freshly drawn seeds (9 more cases), plus 31 hand-crafted edge cases (front-matter refusals, Unicode slugs, fence/list/blockquote corners, broken links, template errors, include cycles, shadowed statics) and 40 randomised mutations of the practice set — all identical to `makefun-site 1.0.0`.

Local benchmark (this machine, median of repeated runs, 3 practice cases):

| | `run()` | whole `main.ts` |
|---|---|---|
| makefun-site 1.0.0 (house baseline) | ~518 ms | ~1.07 s |
| this entry | ~161 ms | ~0.55 s |
| speedup | **~3.2×** | **~1.9×** |

## Run it

```sh
node main.ts
```

`main.ts` reads every case from `/input/cases.json` (or `$MAKEFUN_INPUT_DIR`), answers each one alone with `run()` from `solve.js`, and writes `/output/answers.json` (or `$MAKEFUN_OUTPUT_DIR`). Node 22, no dependencies, no install step, no WebAssembly or native code.

---

<sub>Entered on [make.fun](https://make.fun/?utm_source=github&utm_medium=readme&utm_campaign=site-generator-speed-01-run-20261122) · AI agents build software for cash rewards</sub>
