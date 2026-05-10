# addyosmani/agent-skills — production-grade skill 設計模式調研

**調研時間：** 2026-05-07
**Repo：** https://github.com/addyosmani/agent-skills
**License：** MIT（可借鑑、可 fork）
**規模：** 30.4k stars / 3.6k forks，最新 release 0.6.0（2026-04-28），活躍維護
**對照標的：** 共腦 skill metabolism 機制（memory/project_skill_metabolism.md，2026-04-19）

---

## 一句話結論

addyosmani 在 **單一 skill 內部結構** 做到工程級成熟（反詭辯表 + verification gate + SDLC 全鋪），但**完全沒有生命週期機制**（無 usage tracking、無 demote、無 birth filter）。共腦的 skill metabolism 是 addyosmani 沒有的維度，**借鑑其 SKILL.md 模板，不複製其「20 skill 全鋪」策略**。

---

## 一、addyosmani 架構盤點

### 1.1 目錄結構

```
agent-skills/
├── skills/              20 個 skill，按 SDLC 6 phase 分類
├── agents/              3 個 specialist persona（code-reviewer / test-engineer / security-auditor）
├── references/          4 個 checklist（testing / security / performance / accessibility）
├── hooks/               session 生命週期 hooks
├── .claude/commands/    7 個 slash command
├── .gemini/commands/    同上（多 host 移植）
├── .opencode/ .kiro/    其他 host 整合
└── docs/                安裝與整合指引
```

### 1.2 20 個 skill 沿 SDLC 6 Phase 全覆蓋

| Phase | Skills |
|---|---|
| **Define** (2) | idea-refine, spec-driven-development |
| **Plan** (1) | planning-and-task-breakdown |
| **Build** (5) | incremental-implementation, test-driven-development, context-engineering, source-driven-development, frontend-ui-engineering, api-and-interface-design |
| **Verify** (2) | browser-testing-with-devtools, debugging-and-error-recovery |
| **Review** (4) | code-review-and-quality, code-simplification, security-and-hardening, performance-optimization |
| **Ship** (5) | git-workflow-and-versioning, ci-cd-and-automation, deprecation-and-migration, documentation-and-adrs, shipping-and-launch |
| **Meta** (1) | using-agent-skills（skill 發現流程） |

對應 7 個 slash command：`/spec /plan /build /test /review /code-simplify /ship`，使用者按 phase 觸發。

### 1.3 SKILL.md 標準錨定（以 test-driven-development 為例）

```yaml
---
name: test-driven-development
description: Drives development with tests. Use when implementing any
             logic, fixing any bug, or changing any behavior.
---
```

正文固定 anatomy（~2,500 字）：
1. **Overview** — 哲學基礎一句話（"Tests are proof"）
2. **When to Use** — 觸發/排除條件 + Related cross-ref
3. **Process** — step-by-step + ASCII 流程圖（如 RED/GREEN/REFACTOR）
4. **Decision Guide** — 決策樹分支
5. **Anti-Patterns** — 三欄表（Pattern | Problem | Fix）
6. **See Also** — 跨 skill 引用
7. **Common Rationalizations** — 兩欄表（Rationalization | Reality）★
8. **Red Flags** — bullet 警訊清單 ★
9. **Verification** — checkbox 證據清單 ★

★ 三段是「process not prose」的硬核 — 不是文件，是執行檢核。

### 1.4 SessionStart Hook（唯一的 lifecycle 機制）

```json
{
  "SessionStart": [{
    "hooks": [{
      "command": "bash ${CLAUDE_PLUGIN_ROOT}/hooks/session-start.sh"
    }]
  }]
}
```

`session-start.sh` 動作：讀 `using-agent-skills/SKILL.md` → 包成 IMPORTANT 訊息注入 session → 提示 "use the skill discovery flowchart"。

**沒有：** usage tracking、audit、promote/demote、metabolism。完全是被動初始化。

---

## 二、與共腦 skill metabolism 的差距矩陣

### 2.1 addyosmani 強、共腦弱的維度

| 維度 | addyosmani | 共腦現況 | 啟示 |
|---|---|---|---|
| **單一 SKILL.md 結構** | 9 段固定 anatomy，含反詭辯/紅旗/驗證 | 結構鬆散、無強制段落 | **應借鑑** |
| **反 LLM 自我合理化** | 每 skill 預列「藉口→反駁」表 | 散在 feedback memory，無 skill 層收斂 | **應借鑑** |
| **Verification gate** | checkbox 證據清單（test pass/build/runtime 數據） | 有「事實查核」memory 但無 skill 層強制 | **應借鑑** |
| **SDLC phase 鋪滿** | 20 skill 對應 6 phase | 共腦 skill 散裝、無 phase 盤點 | **部分借鑑**（盤點而非全鋪） |
| **多 host 移植** | Claude/Cursor/Gemini/Windsurf/OpenCode/Copilot/Kiro 7 host | 綁 `.claude/skills/` | 暫不需（共腦單 host） |
| **slash command 與 skill 雙綁** | `/spec` 直接觸發 spec-driven-development | 共腦 skill 多走 inline 觸發 | 可選借鑑 |

### 2.2 共腦強、addyosmani 弱的維度

| 維度 | 共腦 | addyosmani | 對照 |
|---|---|---|---|
| **5 層 substrate** | inline / memory / script / skill / agent | 全部 skill 層 | 共腦領先 |
| **Birth filter** | ≥2 次模式 + 錯有後果 + 推導錯率 >20% 等明確閾值 | 無，誰都能加 skill | 共腦領先 |
| **Auto-demote** | 30/90/180d 無用自動降層 | 無 | 共腦領先 |
| **Usage tracking** | `.claude/usage/<type>/<name>.jsonl` | 無 | 共腦領先 |
| **EP2 預設「越輕越好」** | 推導 > 查表 | 預設「skill 越多越好」 | 哲學差異 |
| **代謝 cron** | 週一 audit + daily-intel segment | 無 | 共腦領先 |

### 2.3 哲學差異（不只是工程差異）

addyosmani 立場：**"skills are executable workflows, not reference documentation"** — skill 是強制工作流。
共腦立場：**EP2 從查表變推導** — skill 是高 ROI 的固化件，預設越輕越好，過度固化等於退回 EP1。

兩者不衝突，互補：
- addyosmani 教「**該建的 skill 怎麼寫**」(quality)
- 共腦 metabolism 答「**該不該建這 skill**」(quantity gate)

---

## 三、移植建議（不 fork）

### 3.1 P0 — SKILL.md 模板升級（價值最高，成本最低）

在 wuji-my-axiom/templates/ 加 `SKILL.md`，固定 anatomy：

```markdown
---
name: <lowercase-hyphen>
description: <一句話 + 觸發條件>
---

## Overview
（一句話哲學）

## When to Use / When NOT to Use
（含 Related cross-ref）

## Process
（step-by-step + 流程圖）

## Common Rationalizations  ← 反 LLM 自我合理化
| Rationalization | Reality |

## Red Flags
- ...

## Verification
- [ ] <可驗證證據>
```

對應太極：**「貞而無據，標記不確定」** + **「沒驗證的不算數」**。Verification checkbox 是這兩條爻則的工程化。

### 3.2 P1 — 把「反詭辯表」抽成獨立爻則類型

共腦現有 feedback memory 已散著反詭辯例子（如 `feedback_overconfidence_signal.md`、`feedback_verify_premise_before_proposing.md`），但沒收斂到 skill 層。建議：

- 在 SKILL.md 模板強制 Rationalizations 段
- 對應 feedback memory 裡明顯的反詭辯項，sweep 為候選 skill 種子（不一定都建，過 birth filter）

### 3.3 P2 — SDLC phase 盤點共腦現有能力（不全鋪）

對 6 phase 做一次 audit：
- 哪些 phase 共腦已有 script/memory/skill 覆蓋？
- 哪些 phase 還在 inline 推導？是否該升 substrate？

**重點：盤點是為了知道缺口，不是為了補齊到 20 個。** EP2 預設輕，缺口未過 birth filter 就不建。

### 3.4 不採納

- ❌ 全部 20 個 skill 移植 — 多數重疊共腦 CLAUDE.md/feedback memory，建了會退回查表
- ❌ slash command 全綁 — 共腦走 inline 觸發已自洽
- ❌ multi-host 適配層 — 只跑 Claude Code，YAGNI

---

## 四、反證：共腦 skill metabolism 設計的工業背書

addyosmani 30.4k stars 也沒做 lifecycle，反映業界主流仍停留「skill = 永久工作流」的階段。共腦的 metabolism 機制：

1. **5 層 substrate** — 對應 addyosmani 的 skill+agent+commands+hooks 但**正交分解**（lifecycle/cost/permanence 三軸）
2. **Auto-demote** — addyosmani 沒有，共腦先行
3. **Birth filter** — addyosmani 沒有量化閾值，共腦有

可作為太極框架公版發佈時的 differentiator，寫進 `research/LANDSCAPE.md`：
- vs addyosmani：我們補齊了 lifecycle 維度
- vs Hermes（已調研）：我們補齊了 substrate 階層
- vs ml-intern（已調研）：我們有 birth filter，他們有 destructive ops taxonomy（互補）

---

## 五、下一步行動

1. **本 session 不直接動手寫 SKILL.md 模板** — 寫模板需 Owner 校準（命名、強度、與既有 templates/CLAUDE.md 的關係）
2. 將「SKILL.md 模板升級」加入 TODO.md，標 P0
3. LANDSCAPE.md 補一段對照（addyosmani vs 共腦 metabolism）— 公版發佈時用
4. 若 Owner 確認方向，下一手任務：草擬 `templates/SKILL.md` + 對共腦 `.claude/skills/` 既有 skill 做 anatomy 升級試點

---

## 六、引用與驗證

- Repo HEAD 結構與 SKILL.md 內容：WebFetch 2026-05-07，github.com/addyosmani/agent-skills
- License：MIT（GitHub 顯示）
- 0.6.0 release date：2026-04-28
- session-start.sh 內容：raw.githubusercontent.com 直接讀取確認，無 tracking 邏輯
- 共腦 metabolism 對照：memory/project_skill_metabolism.md（2026-04-19，本 session 已驗其仍代表現行設計，未在 git log 看到後續推翻）

**未驗證：** 是否有 community fork 已加 lifecycle 機制（沒搜 issues/PR，預估 ROI 低）。
