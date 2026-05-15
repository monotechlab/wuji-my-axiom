# Academic Artifacts

The papers in this directory document the framework's deployment history, failure-mode analysis, and structural patches in formal post-mortem form. They are companion artifacts to the framework itself ([`../README.md`](../README.md)) — read these if you want the *why* and *what we learned the hard way*.

## Files

| File | Language | Type | Length |
|---|---|---|---|
| [`wuji-paper-zh.md`](wuji-paper-zh.md) | **中文（主版）** | 研究筆記 v0.2（2026-05-15）— 原始撰寫語言 | ~36 KB |
| [`wuji-paper.md`](wuji-paper.md) | English | Research note v0.2 — translated from Chinese | ~41 KB |
| [`wuji-saga.md`](wuji-saga.md) | 中文 | 敘事姊妹篇 — 補丁如何誕生的故事版 | ~18 KB |
| [`wuji-review.md`](wuji-review.md) | 中文 | 2026-05-15 critical review，驅動 v0.2 修訂的 11 漏洞 + LOCAL 觀察 | ~9 KB |

## Versioning

| Version | Date | What changed |
|---|---|---|
| v0.1 | 2026-04-22 | EPOCH-2 sealed; PATCH-001 + Wuji Patch deployed |
| **v0.2** | **2026-05-15** | 7-day post-deployment revision: positional vs enforcement structure split; selection-bias floor method; 20% anchor tagged `[unverified]`; three parallel routes for what comes next; LOCAL feedback loop acknowledged |
| v0.3 | open | Will treat: Appendix D reproducibility protocol; selection-bias floor computation (Owner-correction count + commit-revert count); cross-provider migration test |

## Read order

If new to the framework:

1. Start at the root [`../README.md`](../README.md) — what the framework is and how to use it
2. [`../docs/THEORY.md`](../docs/THEORY.md) — the *I-Ching* lineage and the core compression
3. Skim [`../docs/PATCH-001.md`](../docs/PATCH-001.md) and [`../docs/PATCH-002.md`](../docs/PATCH-002.md) — the operational patches
4. **Then read [`wuji-paper.md`](wuji-paper.md) or [`wuji-paper-zh.md`](wuji-paper-zh.md)** — the formal academic write-up
5. Optionally [`wuji-saga.md`](wuji-saga.md) — the human side of the story
6. Optionally [`wuji-review.md`](wuji-review.md) — see the paper being audited against itself

## How to cite

```
Mono and Owner. "Wuji Patch: Grounding the Taichi Framework Against the LLM
Meta-Cognition Failure Assumption." v0.2, 2026-05-15.
https://github.com/monotechlab/wuji-my-axiom/blob/main/papers/wuji-paper.md
```

## License

These artifacts inherit the repository's MIT license ([`../LICENSE`](../LICENSE)). Citation appreciated for academic re-use.
