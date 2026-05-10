# Thunderbolt 多模型框架研究報告

> 調研日期：2026-04-19
> 目的：評估 Mozilla Thunderbolt（thunderbird/thunderbolt）的多模型抽象架構，作為太極框架「器可換」層的設計參考
> 資料來源：GitHub repo（commit 狀態 2026-04-19）、官方 README / architecture.md / AGENTS.md / roadmap.md

---

## 1. Thunderbolt 是什麼

Thunderbolt 是 Mozilla Thunderbird 團隊於 2025-07-23 開源的跨平台 AI 客戶端（MPL-2.0），2026-04 進入 GitHub trending，兩週內 2,000+ stars。

**口號**：AI You Control — Choose your models. Own your data. Eliminate vendor lock-in.

**定位**：企業自架、離線優先、模型無關的 AI chat client。對標閉源 ChatGPT / Claude 桌面 app，主打「資料主權」與「供應商可替換」。

**技術基礎**：
- 前端：React 19 + Vite + Radix UI + Zustand + TanStack Query
- Shell：Tauri（desktop macOS/Linux/Win + mobile iOS/Android）
- 本地資料：SQLite + Drizzle ORM（offline-first source of truth）
- 後端：Elysia on Bun + PostgreSQL + PowerSync（多裝置同步引擎）
- AI 層：**Vercel AI SDK** + **MCP Client**
- 認證：Better Auth（OTP/OIDC），OAuth 整合 Google/Microsoft
- 部署：Docker Compose / Kubernetes 全套可自架

**當前狀態（2026-04-19）**：
- 仍處於 active development、進行中的 security audit
- 主打 enterprise on-prem 客戶
- 尚未完全 offline-first（auth + search 仍依賴 backend）
- 沒有公共 inference endpoint，必須 BYOK（自帶 API key）

---

## 2. 核心架構

### 2.1 四層分界

```
┌──────────────────────────────────────────────┐
│ EXTERNAL（第三方 SaaS，可替換）              │
│   LLM Providers · OAuth · PostHog · Resend   │
└──────────────────────────────────────────────┘
                    ▲
┌──────────────────────────────────────────────┐
│ SERVER（可自架）                              │
│   Backend API (Elysia/Bun)                   │
│   ├─ Auth (Better Auth)                      │
│   ├─ Inference Proxy（Rate limit + Routing） │
│   └─ PowerSync Engine → PostgreSQL           │
└──────────────────────────────────────────────┘
                    ▲
┌──────────────────────────────────────────────┐
│ LOCAL（使用者裝置）                           │
│   Tauri Shell                                │
│   ├─ React UI                                │
│   ├─ State (Zustand/TanStack/Drizzle)        │
│   ├─ AI Chat (Vercel AI SDK + MCP Client)    │
│   ├─ SQLite（offline-first 真實來源）        │
│   └─ E2E Encryption（optional）              │
└──────────────────────────────────────────────┘
```

### 2.2 三個關鍵抽象層

**(a) Vercel AI SDK — 模型 API 抽象**
前端不直接呼叫各家 LLM SDK，而是透過 Vercel AI SDK 統一介面（streaming、tool calls、message format）。切換 Claude/GPT/Mistral 只改 provider config，不動業務邏輯。

**(b) Inference Proxy — 路由與限流**
所有 LLM 呼叫從前端經後端 proxy 轉發到外部 provider。proxy 層負責：
- 模型路由（哪個請求送到哪家）
- Rate limiting（per-user / per-model）
- API key 集中管理（不下發到 client）
- SSE streaming 中繼

**(c) MCP Client — 工具/技能抽象**
已內建 MCP Client（Preview），roadmap 中 Agent Memory / Agent Skills 規劃為「Planned」。代表 Thunderbolt 將 tools 也作為可替換層對待。

### 2.3 OpenAI-compatible 作為 lingua franca

使用者設定中可加入「任何 OpenAI-compatible endpoint」。Ollama / llama.cpp 原生支援此格式，使本地推論與雲端 API 無差別。這是 2026 年業界事實標準。

### 2.4 資料主權設計

- SQLite 為本地 source of truth，後端同步是次要而非必要
- PowerSync 做最終一致性的跨裝置同步（非 realtime）
- 可選 E2E encryption：後端只存 ciphertext
- 整條 stack（backend + PostgreSQL + PowerSync + auth）可用一份 Docker Compose 起起來

---

## 3. 與太極框架的映射

太極框架的「道 / 德 / 器」三層架構與 Thunderbolt 的分層有直接對應：

| 太極框架 | Thunderbolt 對應 | 啟示 |
|---|---|---|
| **道**（不動）CLAUDE.md 太極 + 生生 + 爻則 | AGENTS.md / CLAUDE.md 根目錄協議 | 治理文件不與模型綁定，跑在任何 LLM 上意義不變 |
| **德**（常新）Memory 使用中校準累積 | 本地 SQLite + 可選 E2E 加密 | 個人校準資料必須本地優先、加密可選 |
| **器**（可換）Skills / Projects | Vercel AI SDK + MCP Client + Inference Proxy | 模型與工具都是可插拔的實現層 |

**三條具體啟示**：

**1. 模型抽象不是未來工作，是架構前置**
太極框架目前預設跑在 Claude Code 上。若未來要推廣到 Cursor / Zed / Thunderbolt 本身 / 自架 Ollama，現在就該把「呼叫 AI」抽象為 provider-agnostic 介面。Vercel AI SDK 是 TypeScript 生態的既有答案，Python 生態有 LiteLLM / OpenAI-compatible wrapper 類似地位。

**2. OpenAI-compatible 是事實標準**
冷啟動校準（calibration/）要輸出的 prompt 應該避免 Claude-only 的特性（例如 Anthropic 特有的 system prompt 格式）。遵守 OpenAI chat completions schema 能最大化框架的可攜性。

**3. MCP 作為 skills 的運輸層**
太極框架「器（可換）Skills / Projects」目前是 Markdown + bash script 組合。若要在 Thunderbolt / Cursor / Claude Code 之間共用，把 skills 包成 MCP server 是目前最通用的做法。roadmap 中 Thunderbolt 已規劃 Agent Skills，時間點對齊太極框架的公開發佈。

---

## 4. 可直接借鑑的工程實踐

這些是 AGENTS.md 中 Mozilla 團隊寫給自己（與 AI agent）的工作守則，與太極框架「爻則」層質地相似：

- **Bias towards tasteful simplicity** — 與「重複即冗餘，應當消除或系統化」呼應
- **Prefer optimistic code over defensive code** — 錯誤讓它在開發期大聲爆，不到處 try/catch — 與「沒驗證的不算數」互補
- **Question and recommend alternatives — your goal is better outcomes, not blind execution** — 與「沉默不是確認」同源
- **Soft delete by default** — 資料可逆性，與「可逆的敢賠，不可逆的不能賭」直接對應

AGENTS.md 已成為 GitHub 上 AI-native 專案的新慣例（與 CLAUDE.md 並列）。太極框架公版若推廣，應考慮同時產出 AGENTS.md 格式。

---

## 5. 差異與不可借鑑處

**(a) Thunderbolt 是產品，太極框架是治理協議**
Thunderbolt 解的是「跨模型的 UI 容器」，太極框架解的是「跨模型的價值推導」。前者是器層作品，後者是道德器三層都碰。直接移植 Thunderbolt 的架構會誤把太極框架定位為另一個 chat client。

**(b) Thunderbolt 尚未有 agent 自主行為**
Roadmap 中 Agent Memory / Agent Skills 皆為 Planned。目前仍是 chat mode 為主。太極框架的「生生循環」與「使用中校準」在 agent 自主行為上更前沿。不該用 Thunderbolt 現況反推太極框架的設計。

**(c) 企業 on-prem 定位差異**
Thunderbolt 首要客戶是企業 IT，需求是 FDE（法務/合規）、OIDC、審計。太極框架是一人組織治理，過度搬入 enterprise 特性會破壞輕量性。

**(d) MPL-2.0 vs. 太極框架授權**
Thunderbolt 用 MPL-2.0（copyleft 中間派）。若太極框架公版要引用其程式碼片段，必須處理授權相容性。設計參考無風險，直接 copy code 需評估。

---

## 6. 行動建議

**短期（本週驗證週內）**
- 將「模型抽象」列入 docs/ARCHITECTURE.md 的待補章節，說明太極框架本身不依賴特定 LLM，提及 Vercel AI SDK / LiteLLM / OpenAI-compatible 為實作路徑
- LANDSCAPE.md 新增類別三「多模型 AI client」，收錄 Thunderbolt 與同類方案（Open WebUI、LibreChat、Msty），標記與太極框架的正交關係

**中期（發佈後）**
- 評估把 calibration/ 產出的 CLAUDE.md 同時匯出為 AGENTS.md 格式（簡單的格式轉換）
- 考慮把「我的卦」的冷啟動校準結果包成 MCP server，讓 Thunderbolt / Cursor / 其他 MCP host 皆可載入

**長期（論文後）**
- 若太極框架有 web-based 實作需求，Tauri + React + Vercel AI SDK + Drizzle 這套 stack 是可抄的 baseline，不必從零架

---

## 7. 結論

Thunderbolt 不是太極框架的競品，而是**器層的參考實作**。它證明了一件事：在 2026 年的業界，模型抽象（Vercel AI SDK）、工具抽象（MCP）、資料本地化（SQLite + E2E）已有成熟答案，太極框架不必自己重造。

對太極框架的最大警示：**別把治理協議（道/德）綁死在任何單一 LLM 或 host 上**。現在是 Claude Code，三年後可能是 Thunderbolt、Cursor、甚至尚未出現的工具。器可換的前提是器被正確抽象過。

貞而無據的部分：
- Thunderbolt 的 Inference Proxy 具體路由策略（權重、fallback、model selection）未從 public repo 讀到實作細節，本報告只陳述分層意圖
- Agent Memory / Skills roadmap 為 Planned 狀態，實際設計尚未公開，本報告不猜測其形態

---

**參考連結**
- GitHub: https://github.com/thunderbird/thunderbolt
- Homepage: https://thunderbolt.io
- Architecture: https://github.com/thunderbird/thunderbolt/blob/main/docs/architecture.md
- AGENTS.md: https://github.com/thunderbird/thunderbolt/blob/main/AGENTS.md
- Roadmap: https://github.com/thunderbird/thunderbolt/blob/main/docs/roadmap.md
