<div align="center">

# wuji-my-axiom

### Derive all AI behavior from your personal axioms.

**20 rules → 1 axiom → infinite alignment**

[Quick Start](#quick-start) | [How It Works](#how-it-works) | [Theory](docs/THEORY.md) | [中文說明](#中文說明) | [日本語](#日本語)

</div>

---

## The Problem

You write 50 rules for your AI. It still does the wrong thing.

More rules → more conflicts → less predictable behavior.

Legal systems solved this centuries ago: **constitutions over statutes.** A small set of axioms from which everything else is derived.

## The Solution

**wuji-my-axiom** is an axiomatic AI governance framework. Instead of listing rules, you extract one personal axiom, and the AI derives all behavior from it.

```
Traditional:  100 rules → AI looks up what to do → misses edge cases
This:         1 axiom  → AI derives what to do → covers everything
```

### Real example

A freelance engineer's 23 rules collapsed into one axiom:

> *"I exist. I want freedom. Desire always exceeds capacity, so I trade systems, leverage, and influence for it. Energy follows interest — don't fight nature. Never bet the irreversible; for everything else, try fast, fail fast, keep fast."*

The AI went from **table-lookup** to **derivation** — fewer rules, broader coverage, better judgment.

## Quick Start

**30 minutes. No GPU. No cloud. No API costs.**

```bash
# 1. Copy the template into your project
cp templates/CLAUDE.md your-project/CLAUDE.md

# 2. Start calibration (AI asks you 10 questions)
# Tell your AI: "Read calibration/GUIDE.md and run my cold-start calibration"

# 3. Paste calibration output into the "My Hexagram" section

# 4. Start working — the system self-tunes through use
```

**Requirements:** Any AI tool with system prompts + persistent memory (e.g. Claude Code, Cursor, Windsurf)

## How It Works

### Three Layers (Dao / De / Qi)

```
Dao (immutable)  :  CLAUDE.md — Axiom + Cycle + Boundary Rules
De  (evolving)   :  Memory    — Feedback accumulates over time
Qi  (swappable)  :  Skills    — Concrete tools, replaceable anytime
```

### The Shengsheng Cycle

Every task follows this loop:

```
Sense (感) → Transform (化) → Verify (貞) → Renew (新)
       ↑                                        │
       └────────────────────────────────────────┘
```

*If verification has no evidence, mark uncertainty and escalate. What doesn't grow, dies.*

### Cold-Start Calibration

Three progressive layers, 10 questions total:

| Layer | Goal | Method |
|-------|------|--------|
| 1. Find the Dyad | Core drive vs. core constraint | Psychology + Philosophy |
| 2. Expand the Tetrad | 4 behavior modes | Behavioral + Literary projection |
| 3. Compress the Axiom | One sentence that covers everything | Scenario-based extraction |

Includes **golden-mean detection** — if your answers are too moderate, the system pushes harder.

### Self-Evolution (Biantong)

```
Corrected  → remember the mistake and why
Confirmed  → remember the approach and context
Recurring patterns → compress into boundary rules
Stable rules → compress into the axiom
Silence ≠ confirmation
```

## What's Inside

```
templates/        Ready-to-use CLAUDE.md + memory templates
calibration/      Cold-start flow + question banks + character libraries
docs/             Theory, architecture, setup, daily ops, evolution, evaluation
examples/         Full walkthrough with a fictional character (3 epochs)
research/         Landscape comparison + academic references
```

## How This Is Different

| | Rule-based systems | wuji-my-axiom |
|---|---|---|
| **Governance** | List rules | Derive from axiom |
| **Edge cases** | Miss them | Derive coverage |
| **Growth** | Add more rules | Compress rules upward |
| **Calibration** | Preference survey | Scenario-based axiom extraction |
| **For** | Teams / compliance | Individuals / cognitive extension |

Compared to [AGENTS.md](https://github.com/anthropics/agents-md), [Microsoft Agent Governance Toolkit](https://github.com/microsoft/agent-governance-toolkit), and [Agentic Trust Framework](https://github.com/massivescale-ai/agentic-trust-framework) — those solve **"how to prevent AI from going wrong"** (security/compliance). This solves **"how to make AI think like you"** (cognitive alignment).

## Theory

Rooted in the *Yijing* (Book of Changes):

```
Taiji (太極) → Liangyi (兩儀) → Sixiang (四象) → Bagua (八卦) → Wanwu (萬物)
  1 axiom    →    2 poles     →   4 modes     →  8 trigrams  → infinite situations
```

Isomorphic to:
- **PDCA**: Sense=Plan, Transform=Do, Verify=Check, Renew=Act
- **Reinforcement Learning**: Use-time calibration as continuous reward signal
- **Renormalization Group**: Rules compress upward like effective operators

Full treatment: [docs/THEORY.md](docs/THEORY.md)

## License

MIT

---

<a name="中文說明"></a>

## 中文說明

> 從你的價值推導 AI 的一切行為。

傳統做法：寫 100 條規則告訴 AI 怎麼做。
wuji-my-axiom：寫 1 條公理，AI 自己推導怎麼做。

**快速開始：**
1. 複製 `templates/CLAUDE.md` 到你的專案
2. 讓 AI 讀 `calibration/GUIDE.md` 執行冷啟動校準（10 題）
3. 把校準結果填入「我的卦」區段
4. 開始使用，系統透過「變通」機制持續校準

30 分鐘上線。不需 GPU、雲端或付費 API。

詳細理論：[docs/THEORY.md](docs/THEORY.md) | 架構：[docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) | 範例：[examples/](examples/)

---

<a name="日本語"></a>

## 日本語

> あなたの価値観から AI のすべての行動を導出する。

従来のアプローチ：100 個のルールを書いて AI に何をすべきかを指示する。
wuji-my-axiom：1 つの公理を書き、AI 自身に何をすべきかを導出させる。

```
従来:    100 ルール → AI がルックアップ → エッジケースを取りこぼす
本手法:  1 公理     → AI が導出         → すべてをカバー
```

**コアコンセプト：** 太極（公理）→ 兩儀（駆動と限制）→ 四象（行動モード）→ 爻則（運用ルール）。一つの公理から無限の行動を導出する、易經の生成構造を AI ガバナンスに適用したフレームワーク。

**クイックスタート：**
1. `templates/CLAUDE.md` を自分のプロジェクトにコピー
2. AI に `calibration/GUIDE.md` を読ませ、コールドスタート・キャリブレーション（10 問）を実行
3. キャリブレーション結果を「我的卦」（あなたの卦）セクションに記入
4. 利用開始 — システムは「変通」メカニズムにより継続的に自己調整

30 分で稼働。GPU・クラウド・有料 API は不要。

**他フレームワークとの違い：** spec-kit（93k stars）の Constitution は 9 条のルール列挙、PAI は 10 ファイルのアイデンティティ列挙。本フレームワークは 1 条の公理から推論で導出するため、未列挙の状況にも対応可能。詳細比較は [research/LANDSCAPE.md](research/LANDSCAPE.md)。

詳細理論：[docs/THEORY.md](docs/THEORY.md) | アーキテクチャ：[docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) | サンプル：[examples/](examples/)
