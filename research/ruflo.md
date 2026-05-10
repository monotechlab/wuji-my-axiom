# ruvnet/ruflo 調研

> 調研日 2026-05-07 ｜ License: MIT ｜ Repo: github.com/ruvnet/ruflo
> 任務：萃取多 agent swarm orchestration 設計，找出可移植到共腦 worker dispatch 的部分

## 一句話結論

ruflo 是「企業版 Claude Code orchestration 平台」，核心抽象 = Queen + Topology + Hooks + 三層記憶 + 五階學習迴路。**整套體量過大不適合一人組織直接 fork**，但其 (1) Queen 類型分類 (2) 五階學習迴路 (3) 生命週期 hook taxonomy (4) capability-based dispatch — 四項可直接概念移植到共腦 worker dispatch，無需引入 ruflo 本體。

## 來源憑據

- README + USERGUIDE.md（284KB / 7614 行）— `raw.githubusercontent.com/ruvnet/ruflo/main/docs/USERGUIDE.md`
- Wiki: Agent-System-Overview / Agent-Categories
- GitHub repo metadata: TS 65.8% / JS 21.5% / Python 7.8% / Shell / Rust
- v3.7 為當前 production-ready release（5800+ commits, 10 個月）

未驗證項：每項數字宣稱（89% routing accuracy / 150x faster search / 0.12ms per insight）皆來自 README 自我宣傳，未在源碼或第三方 benchmark 復核。

## 架構摘要

```
User → Claude Code/CLI
  → Orchestration Layer (MCP Server, Router, ~27 Hooks)
    → Swarm Coordination (Queen, Topology, Consensus)
      → 60-100+ Specialized Agents
        → Memory & Learning (AgentDB, HNSW, SONA, ReasoningBank)
          → LLM Providers (Claude, GPT, Gemini, Cohere, Ollama)
```

技術棧重點：Node.js MCP server 殼 + Rust AI engine 核 + HNSW 向量檢索 + 可選 MongoDB / Supabase。

## 四個核心抽象（細節）

### 1. Queen 類型（dispatch 角色）

| 類型 | 用途 | 對應共腦情境 |
|---|---|---|
| Strategic Queen | 高層目標協調，研究/規劃 | Mono 在 CONTEXT.md 規劃時 |
| Tactical Queen | 直接任務管理，實作 | Mono 派 worker 跑特定 task |
| Adaptive Queen | 即時策略調整，優化 | Mono 處理多焦點專案切換 |

**啟示**：共腦目前一律是 tactical 派工。**沒有 strategic 模式的明確區分** — Mono 在規劃週焦點時，仍然用「直接派 worker」的 tactical 心智，導致週焦點重排決策做得草率。

### 2. Topology（協調拓撲）

- **Hierarchical**（樹狀，queen 領導）— 共腦現況
- **Mesh**（peer-to-peer，工人之間直接通訊，容錯）— 共腦無
- **Ring / Star / Hybrid / Adaptive**

**啟示**：共腦 worker 之間完全無通訊（透過 git 間接同步），目前對單 Owner 一次一焦點足夠；當 worker 並發 ≥3 且任務有依賴時，會浪費 token 重複 git pull。但**這不是設計缺陷而是 KISS 選擇** — mesh 帶來的協調成本對單 Owner 是負收益。

### 3. Hooks Taxonomy（生命週期事件）

USERGUIDE 列出的 hook 類別（27→33 之間，因 CLI 與 MCP hook 計數不同）：

```
session:start, session:end
agent:pre-spawn, agent:post-spawn, agent:pre-terminate
task:pre-execute, task:post-complete, task:error
memory:pre-store, memory:post-store, memory:pre-retrieve
pre-edit, post-edit, pre-command, post-command
route, explain, pretrain
intelligence/*, worker/*, progress
```

**啟示**：共腦 `start-worker.sh` 目前 hook 點不齊。對照 ruflo 列表，共腦缺：

| Hook | 共腦現況 | 可加什麼 |
|---|---|---|
| `task:error` | 無 — worker 失敗只留 .worker-result.json status=BLOCKED | hook 寫進 brain-events.log + 觸發 nudge |
| `task:post-complete` | 有（.worker-result.json）但 Mono 不主動讀 | Monitor 工具掃 .worker-result 即時通知（已在 research/claude-monitor-tool.md 規劃，未實作）|
| `agent:pre-spawn` | 無 — start-worker.sh 直接 tmux | 加 lesson-scan.sh "<topic>"（已有腳本，但未強制注入 prompt）|
| `memory:pre-retrieve` | 無 | 啟動校準時自動掃 MEMORY.md 相關 entry（已在 boot 流程裡）|

最高 ROI：**`agent:pre-spawn` 注入 lesson** — 已有 `scripts/lesson-scan.sh`，把它從可選變強制就完工。

### 4. 五階學習迴路

ruflo 文件原文：`RETRIEVE → JUDGE → DISTILL → CONSOLIDATE → ROUTE`

對照共腦「生生」`感 → 化 → 貞 → 新`：

| ruflo | 生生 | 共腦現況 |
|---|---|---|
| RETRIEVE | 感 | MEMORY.md + CONTEXT.md 載入 |
| JUDGE | 化 | Mono 推導 |
| DISTILL | 貞 | 不確定就說不確定 |
| CONSOLIDATE | （新的一部分）| feedback memory 寫入 |
| ROUTE | 新（迴向）| 派 worker / 更新 TODO |

**啟示**：ruflo 的五階比生生四階多了一階 **CONSOLIDATE**（將 episodic → semantic 壓縮）。共腦目前**有零散的記憶寫入但無定期壓縮機制** — `feedback_*.md` 越積越多，沒有自動「升級為爻則」的觸發點。CLAUDE.md 寫了「定期：重複的模式升級為爻則」但這是 manual 動作，從未發生過。

→ 可移植機制：**weekly memory consolidation cron** — 掃 memory/feedback_*.md 找重複模式，提名升爻則，等 Owner 拍板。

## 三層記憶（不可全盤移植）

ruflo: Working (1MB, LRU) / Episodic (importance ranking) / Semantic (consolidated)
共腦: 熱（自動載入）/ 溫（按需）/ 冷（archive）

**結論：拓撲一致，實作差異大**。ruflo 用 HNSW 向量檢索 + AgentDB；共腦用 markdown index + grep。Owner 的記憶量級（~80 個 memory entry）**遠不到向量檢索的 break-even 點**。grep 索引在 100 entry 內仍是最低延遲方案。**不引入向量庫**。

## 不可移植 / 不該移植

1. **100+ agent 庫**（coder/tester/reviewer/architect/security/...）— 共腦是一人組織，agent 多≠好。Owner 自己就是 architect+reviewer，子 agent 只在「需要平行/隔離 context」時才有意義。
2. **Consensus algorithms**（Majority / Weighted / Byzantine）— 單 Owner 無投票場景。
3. **Federation Layer**（zero-trust 跨組織協作）— 共腦是 single-org。
4. **AgentDB / HNSW 向量檢索** — 量級不到。
5. **MCP Server 215 tools** — Claude Code 已內建 + Owner 自寫 scripts/，無需再裝一層。

## 移植清單（按 ROI 排序）

### P0 — 立即可做（成本 < 30 min）

- [ ] **強化 `agent:pre-spawn` hook**：在 `scripts/start-worker.sh` 派 worker 前自動跑 `lesson-scan.sh "<topic>"`，把結果注入 worker prompt
  - Why: 已有腳本，只差「強制」；對齊 ruflo agent:pre-spawn pattern
  - 風險：低，可逆

### P1 — 本週可做

- [ ] **Queen 類型語意化**：在共腦啟動校準（boot 流程）加一個 mode flag —
  - `mono --strategic`：規劃週焦點時用
  - `mono --tactical`：派 worker 時用（預設）
  - `mono --adaptive`：跨專案切換時用
  - Why: 強迫 Mono 區分高層規劃 vs 直接派工，避免 strategic 決策被 tactical 慣性壓掉
  - 風險：低，純認知 framing 差別
  - **疑慮**：可能只是裝飾，需驗證是否真的影響行為

- [ ] **`task:error` hook**：worker 寫 .worker-result.json status=BLOCKED 時，自動 append 到 brain-events.log + 觸發 last-nudge 機制
  - Why: 目前失敗訊號需要 Mono 主動讀 .worker-result.json，event-driven 化
  - 與 research/claude-monitor-tool.md 的 Monitor 規劃合併實作

### P2 — 等驗證後再做

- [ ] **Weekly memory consolidation cron**：每週日掃 memory/feedback_*.md，找出 3 次以上類似模式，產出「建議升爻則」清單
  - Why: 對應 ruflo CONSOLIDATE 階段；CLAUDE.md 的「定期壓縮」目前是空頭支票
  - 風險：低（只產建議，由 Owner 拍板）；但易產生噪音，需先觀察 feedback memory 的實際增長率

### 不做

- 向量檢索：量級不到
- mesh topology：worker 並發未到 ≥3
- agent 庫擴增：Owner 是全才
- consensus 機制：單 Owner

## 對 taichi-framework 公版的影響

可在 `research/LANDSCAPE.md` 加入 ruflo 為「企業級多 agent orchestration」的代表，與共腦的「一人組織治理」對照：**ruflo 解決多人/多組織協同的協調成本，太極框架解決一人組織的認知超載** — 不同問題、不同最佳設計點。

ruflo 的存在反向佐證太極框架定位正確：**80% 的 ruflo 機制（共識/聯邦/100+ agent）對一人組織是負收益**，太極框架用「公理推導」取代「規則列舉與多 agent 投票」是正確的簡化方向。

## 爻則門檻檢查

1. 與 CONTEXT.md / contracts/ 矛盾？— 無
2. 來源是常識而非從專案需求推導？— 移植清單每項都對齊共腦現況推導，未照搬 ruflo
3. 錯了不可逆？— P0/P1 皆可逆（只是腳本與 framing 改動），P2 weekly consolidation 若噪音多可關
