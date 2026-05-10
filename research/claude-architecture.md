# Claude Design 四層解剖研究報告

> 調研日期：2026-04-21
> 目的：系統性理解 Anthropic Claude 生態的設計哲學（模型家族／工具／框架／治理），作為太極框架「道—德—器」三層架構的對照與設計參考
> 資料範圍：Anthropic 官方文件、arxiv 2604.14228（Dive into Claude Code）、arxiv 2212.08073（Constitutional AI）、2026 年技術部落格

---

## 1. 四層架構概觀

Claude 不是一個模型，是一個**跨四層的生態**。任何想理解 Claude design 的人，必須同時看懂這四層：

```
┌──────────────────────────────────────────────┐
│ 層 4：Constitutional AI（價值／治理）          │
│   Helpful · Honest · Harmless                 │
│   公理式自我對齊（RLAIF）                     │
├──────────────────────────────────────────────┤
│ 層 3：Agent SDK（框架）                        │
│   tool loop + streaming + MCP                 │
│   窄授權 specialized agents（反通用 agent）   │
├──────────────────────────────────────────────┤
│ 層 2：Claude Code（工具／工程）                │
│   while-loop core                             │
│   5 層擴展：MCP / Plugins / Skills /          │
│             Subagents / Hooks                 │
│   7 模式 permission + ML classifier           │
├──────────────────────────────────────────────┤
│ 層 1：模型家族（產品）                         │
│   Haiku · Sonnet · Opus                       │
│   intelligence × speed × cost 三維頻譜         │
└──────────────────────────────────────────────┘
```

---

## 2. 層 1：模型家族（產品層）

**三檔分層**：Haiku（快）／ Sonnet（平衡）／ Opus（最強）。自 Claude 3 起維持此三檔設計。

**代數演進**：3 → 4 → 4.5 → 4.6 → 4.7。Opus 4.7 於 2026-04-16 發布，主打：
- Agentic coding 的階躍提升
- 長跨度自主任務（long-horizon autonomous）
- 高解析視覺

**產品哲學**：不提供「一個最強模型」，而是一個頻譜，讓應用依 **intelligence × speed × cost** 三維自選。設計啟示：**單一最大模型不是最優產品形態**，分層是必要的。

---

## 3. 層 2：Claude Code（工具／工程層）

### 3.1 Core

**一個 while-loop**（model → tool → repeat）。絕大多數程式碼不在 loop 本身，而在**環繞 loop 的系統**。

### 3.2 五層擴展機制

| 層 | 用途 | 可呼叫誰 | 性質 |
|---|---|---|---|
| **MCP** | 外部工具的通用 adapter | — | 最底層 |
| **Skills** | Markdown 定義的可重用工作流 | MCP / Subagent / bash | 核心行為 |
| **Subagents** | 隔離 context 的專門化 agent | Skills / MCP（受限） | 防 context poisoning |
| **Hooks** | Lifecycle event 觸發器 | Skills / bash / HTTP | **強制性非勸告性** |
| **Plugins** | 封裝 Skills + Hooks 可分發 | — | 打包層 |

### 3.3 周邊系統

- **7-mode permission system + ML classifier** — per-action 安全分類
- **5 層 context compaction pipeline** — context management
- **Append-only session storage** — 不可覆寫的事件日誌
- **Deferred tool loading（2026）** — MCP 載入 tool 名稱，schema 按需取得

### 3.4 組合規則

- Skills 可呼叫：MCP / Subagents / bash
- Subagents 可呼叫：Skills / MCP（受限）
- Hooks 可觸發：Skills / bash / HTTP
- MCP servers **不**繼承 Read/Write/Bash，除非明示給予

### 3.5 使用模式口訣

**記憶（CLAUDE.md）→ 工作流（Skills）→ 委派（Subagents）→ 強制（Hooks）**

---

## 4. 層 3：Agent SDK（框架層）

### 4.1 核心信念

**「給 agent 一台電腦」**讓它像人類工作。Claude Code 的 tool loop、agent loop、context management 皆可透過 Agent SDK 程式化使用（Python / TypeScript）。

### 4.2 反直覺設計

**不是給你一個通用 agent，而是讓你建多個 specialized agents**，每個窄授權、受限工具、聚焦 system prompt。這與 AutoGPT / 早期通用 agent 框架相反。

### 4.3 提供 vs 刻意不提供

| 提供 | 刻意不提供（留給 app 層） |
|---|---|
| Tool loop | Durable state |
| Streaming | Cost governance |
| Message history | Circuit breakers |
| MCP 一等公民 | Permission scoping |

**設計哲學**：框架處理共通機制，**不替應用決定策略**。

---

## 5. 層 4：Constitutional AI（價值／治理層）

### 5.1 HHH 核心

**Helpful · Honest · Harmless**。不只是 guideline，是訓練目標。

### 5.2 運作機制

兩階段訓練：
1. **SL 階段**：模型用 constitution 批判、修改自己的輸出
2. **RL 階段**：用 AI 評估（非人類）產生 harmlessness preference data → RLAIF

### 5.3 Constitution 來源

三合一：
- UN 人權宣言（挑廣泛共識條款）
- ToS 最佳實踐
- 研究試錯發現的原則

### 5.4 演進

2026-01 Anthropic 發布新版 Claude constitution（持續迭代）。

### 5.5 與 RLHF 的差異

RLHF 依賴人類標註 → 成本高、瓶頸明顯。RLAIF 讓 AI 用公理自評 → **可擴展、可審計**。

---

## 6. 五大驅動價值（arxiv 2604.14228）

Claude Code 設計空間論文識別的五個核心人類動機：

1. **Human decision authority** — 人類決策權
2. **Safety and security** — 安全與保障
3. **Reliable execution** — 可靠執行
4. **Capability amplification** — 能力放大
5. **Contextual adaptability** — 情境適應性

這五項不是 feature，是**設計約束**。Claude Code 的每個架構決策都可回溯到至少一項。

---

## 7. 對太極框架的映射

### 7.1 概念對應表

| 太極框架 | Claude 對應 | 關係性質 |
|---|---|---|
| 太極（道）公理 | Constitutional AI 的 constitution | **同構** — 都用公理自我對齊 |
| 生生循環 感→化→貞→新 | while-loop（model→tool→repeat） | **同構** — 最小核心 + 周邊系統 |
| 爻則 | Permission 7 模式 + ML classifier | Claude 工程化，太極規範化 |
| 德（Memory） | CLAUDE.md + append-only session storage | 記憶分層一致 |
| 器（Skills/Projects） | MCP + Skills + Subagents + Hooks | **Claude 已五層分解，太極仍混合** |
| Worker = 手 | Agent SDK「窄授權 specialized agents」 | **同構** — 反通用 agent |
| 冷啟動校準 | System prompt / CLAUDE.md 初始化 | 太極更深（公理層），Claude 更淺（規則層） |

### 7.2 三條可直接用的啟示

#### 啟示 1：「器」層該參照 Claude 五層分解

太極框架目前把 Skills / Projects / scripts 混在一起。Claude 已驗證 **四軸正交**：

- **MCP**（外部介面抽象）
- **Skills**（工作流定義）
- **Subagents**（context 隔離）
- **Hooks**（強制性自動化）

**行動**：公版 templates/ 應拆成這四類，各自獨立目錄，各有最小範例。

#### 啟示 2：Hook 是「爻則」的工程化

爻則目前靠 AI 勸告自己遵守 — 這是 Wuji Patch 專案已識別的失敗模式（元認知假設失敗）。

Claude 的 Hook 說明：**強制性規則必須靠 deterministic 腳本**，不能靠 prompt。

**行動**：太極框架該區分兩類爻則：
- **勸告型爻則**（留在 CLAUDE.md，靠模型自律）
- **強制型爻則**（落到 hooks，靠 shell script 強制）

這直接回應 feedback memory 中的「forcing function」需求。

#### 啟示 3：Constitutional AI 是公理推導的工業先例

太極框架核心主張「AI 從查表變推導」，在 2022 已由 Anthropic 的 RLAIF 論文（arxiv 2212.08073）驗證。

**行動**：research/LANDSCAPE.md 新增類別「公理式 AI 治理的學術/工業先例」，把 Anthropic 定位為**工業界前輩（非競品）**，強化太極框架的理論合法性。具體：
- 引用 Constitutional AI 作為「公理→行為」可實作的證據
- 指出差異：Anthropic 是訓練時嵌入憲法，太極框架是推論時投射公理 — 兩者互補不互斥

---

## 8. 差異與不可借鑑處

### 8.1 規模差異

Claude 是千億參數模型訓練工程；太極框架是個人 prompt 治理協議。直接類比會誤判工程複雜度。

### 8.2 治理方向相反

- **Claude**：Anthropic 定義 constitution，使用者繼承
- **太極框架**：使用者自己冷啟動校準出 constitution

太極的「個人公理」無法套用 Anthropic 的 collective constitution 設計（雖然後者的 collective CAI 論文值得看）。

### 8.3 Agent SDK 的「不提供清單」是太極框架的必經功課

Agent SDK 刻意不提供 durable state / cost governance / circuit breakers。太極框架若要做 runtime，這些**都得自己設計**，不能等 SDK 給。

---

## 9. 結論

Claude design 不是單點技術，是**從價值（HHH）→ 框架（Agent SDK）→ 工具（Claude Code）→ 產品（模型家族）的完整垂直整合**。

對太極框架的最大啟發：**四層都要設計，缺一層就漏**。
- 價值層：太極公理（已有雛形）
- 框架層：生生循環 + worker 協議（設計中）
- 工具層：Skills / Hooks / MCP 對接（缺口最大）
- 產品層：公版 repo + 冷啟動校準（進行中）

Claude 證明了一條路徑可行。太極框架的價值不在重造這條路徑，在於**把終端使用者從消費者（繼承 Anthropic constitution）變成創作者（推導自己的 constitution）**。

### 貞而無據的部分

- arxiv 2604.14228 列出「六個未來開放設計方向」，但 abstract 未全列；需讀 full paper 才能評估與太極框架的重疊
- Anthropic 2026-01 新版 constitution 的具體條款未在本次調研中取得原文
- Claude Code 的 5 層 compaction pipeline 細節未見公開文件，只有論文引述

---

**參考連結**

- [arxiv 2604.14228 · Dive into Claude Code: The Design Space of Today's and Future AI Agent Systems](https://arxiv.org/abs/2604.14228)
- [arxiv 2212.08073 · Constitutional AI: Harmlessness from AI Feedback](https://arxiv.org/abs/2212.08073)
- [Anthropic · Claude's Constitution](https://www.anthropic.com/news/claudes-constitution)
- [Anthropic · Building agents with the Claude Agent SDK](https://claude.com/blog/building-agents-with-the-claude-agent-sdk)
- [Claude API · Agent SDK overview](https://platform.claude.com/docs/en/agent-sdk/overview)
- [Claude API · Models overview](https://platform.claude.com/docs/en/about-claude/models/overview)
- [alexop.dev · Understanding Claude Code's Full Stack](https://alexop.dev/posts/understanding-claude-code-full-stack/)
- [penligent.ai · Inside Claude Code Architecture](https://www.penligent.ai/hackinglabs/inside-claude-code-the-architecture-behind-tools-memory-hooks-and-mcp/)
- [MarkTechPost · Claude Opus 4.7 Release](https://www.marktechpost.com/2026/04/18/anthropic-releases-claude-opus-4-7-a-major-upgrade-for-agentic-coding-high-resolution-vision-and-long-horizon-autonomous-tasks/)
