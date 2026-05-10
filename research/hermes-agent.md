# Hermes Agent 框架研究報告

> 調研日期：2026-04-12
> 目的：評估 Hermes Agent 框架與太極框架的關聯性、整合可能性

---

## 1. Hermes Agent 是什麼

Hermes Agent 是 Nous Research 於 2026-02-26 發布的開源（MIT）自我改進 AI agent 框架。兩個月內累積 47,000+ GitHub stars。核心定位：**跑在你自己伺服器上的持久 AI 助理，越用越強**。

與其他 agent 框架的關鍵差異：它不只是執行任務，而是從任務中提取可複用的知識（skills），下次遇到類似問題直接調用，形成正向飛輪。

**技術基礎：**
- 底層模型：Hermes 3（基於 Meta Llama 3.1），但可切換任意 LLM（OpenAI、Anthropic、OpenRouter 200+ 模型）
- 語言：Python
- 授權：MIT
- 部署：$5 VPS 即可跑，支援 Docker / SSH / Daytona / Singularity / Modal

**來源：**
- [GitHub](https://github.com/nousresearch/hermes-agent)
- [官方文件](https://hermes-agent.nousresearch.com/)

---

## 2. 核心架構

### 2.1 三層架構

```
┌─────────────────────────────────────┐
│  Interface Layer                     │
│  CLI / Telegram / Discord / Slack    │
│  WhatsApp / Signal                   │
├─────────────────────────────────────┤
│  Agent Core                          │
│  AIAgent (run_agent.py)              │
│  ├── prompt_builder.py  系統提示組裝 │
│  ├── context_engine.py  上下文引擎   │
│  ├── context_compressor.py 有損摘要  │
│  ├── prompt_caching.py  提示快取     │
│  └── auxiliary_client.py 輔助 LLM    │
├─────────────────────────────────────┤
│  Execution Layer                     │
│  model_tools.py  工具發現/調度       │
│  toolsets.py     工具分組/預設       │
│  hermes_state.py SQLite+FTS5 狀態庫  │
│  40+ 內建工具                        │
└─────────────────────────────────────┘
```

### 2.2 記憶系統（三層）

| 層級 | 機制 | 儲存 | 特性 |
|------|------|------|------|
| Layer 1: 持久記憶 | MEMORY.md + USER.md | ~/.hermes/memories/ | 每 session 注入 system prompt，~2200 字元上限，agent 自主管理增刪 |
| Layer 2: 技能庫 | Markdown 技能文件 | ~/.hermes/skills/ | 從成功任務自動提取，可搜尋，會自我改進 |
| Layer 3: 會話搜尋 | SQLite FTS5 全文索引 | ~/.hermes/state.db | ~10ms 搜尋延遲，跨 session 回溯，LLM 摘要壓縮 |

**可選：Honcho AI-Native Memory** — 向量嵌入 + 辯證推理的無界跨 session 使用者建模。

**關鍵設計決策：**
- **Frozen Snapshot Pattern**：system prompt 在 session 開始時擷取一次，中途不更新。記憶寫入立即存磁碟但下次 session 才生效。原因：保護 LLM prefix cache。
- **字元上限強制**：記憶滿時 agent 自動合併/替換，不無限膨脹。
- **安全掃描**：記憶寫入前掃描 prompt injection / 憑證外洩模式。

### 2.3 自我改進學習迴圈

```
任務完成（≥5 tool calls）
    → 模式提取（分析步驟）
    → 技能創建（寫 Markdown skill 文件）
    → 技能改進（下次使用時根據結果修正）
    → 週期性自省（每 15 任務評估整體表現）
```

**Periodic Nudge 機制**：固定間隔發送內部系統提示，讓 agent 回顧近期行為，評估是否有值得持久化的知識。不需使用者觸發，自動運行。

---

## 3. 與其他 Agent 框架的差異

| 維度 | Hermes Agent | OpenClaw | CrewAI | AutoGPT | Claude Code |
|------|-------------|----------|--------|---------|-------------|
| **核心理念** | 個人助理，越用越強 | SOUL.md 設定式治理 | 多 agent 角色協作 | 自主目標達成 | IDE 內結對程式 |
| **記憶** | 三層持久記憶 + 技能庫 | 功能性但扁平 | Session 內 | 有限持久化 | CLAUDE.md + auto memory |
| **自我改進** | 內建學習迴圈 | 無 | 無 | 有限 | 無（靠 memory 手動累積） |
| **多平台** | 6 通道統一閘道 | CLI 為主 | Python API | CLI/Web | CLI + IDE |
| **模型鎖定** | 無（任意 LLM） | 低 | 低 | 低 | Claude only |
| **部署成本** | $5-50/月 | 依賴 API 費 | 依賴 API 費 | 依賴 API 費 | $59-200/月 |
| **GitHub Stars** | 47k（2 個月） | 345k+ | ~80k | ~170k | N/A |
| **成熟度** | 早期（v0.7） | 穩定 | 穩定 | 穩定 | 穩定 |

**關鍵差異點：**
1. **Hermes 是唯一內建學習迴圈的框架** — 其他框架的記憶是事實型的，Hermes 的記憶是程序型的（記住方法，不只記住事實）
2. **Hermes 不鎖定模型** — Claude Code 只能用 Claude，Hermes 可換任意 LLM
3. **Hermes 定位是通用 agent** — Claude Code 專注編程，CrewAI 專注多 agent 協作

---

## 4. 能否用於共腦架構？

### 4.1 共腦現有架構

```
Owner ←→ Mono（Claude Code session）
              ├→ Worker（背景 tmux agent）
              ├→ Memory（file-based, MEMORY.md 索引）
              └→ RPi5 永遠在線
```

### 4.2 Hermes 與共腦的對齊點

| 共腦概念 | Hermes 對應 | 對齊程度 |
|---------|------------|---------|
| 太極（公理推導） | **無對應** | ❌ 無 |
| 生生循環（感→化→貞→新） | 學習迴圈（任務→提取→技能→改進） | 🔶 結構類似但語義不同 |
| 爻則（邊界規則） | MEMORY.md 中的行為規則 | 🔶 部分對齊 |
| 變通（被糾正/確認→記住） | Periodic Nudge + 技能改進 | 🔶 機制類似 |
| Worker（背景執行） | 多終端後端（Daytona/Modal） | ✅ 高度對齊 |
| Memory 系統 | 三層記憶 | ✅ 結構更完善 |

### 4.3 關鍵差距

1. **沒有公理治理層**：Hermes 的行為由 MEMORY.md 中的規則列舉驅動，不是公理推導。這正是太極框架要解決的問題——Hermes 會遇到和 LANDSCAPE.md 中其他框架一樣的「未列舉情境無法推導」問題。
2. **技能是程序型的，不是治理型的**：Hermes skill = 「怎麼做某件事」，太極框架 = 「為什麼這樣做」。兩者互補但不同層。
3. **學習迴圈缺乏方向錨點**：Hermes 的自我改進是無方向的——沒有太極作為改進的錨，可能漂移（與 LANDSCAPE.md 對 ChristopherA 的評價相同）。

### 4.4 整合可能性

**方案 A：Hermes 作為共腦的「器」層**

```
道（太極框架 CLAUDE.md）→ 治理方向
德（Memory）            → 持續校準
器（Hermes Agent）      → 執行引擎 + 技能累積
```

可行性：🔶 **中等**。Hermes 可作為 Worker 的執行層，但需要：
- 將太極公理注入 Hermes 的 system prompt（替換其 MEMORY.md 行為規則）
- 改造學習迴圈讓新 skill 受公理約束
- Hermes 目前不支援 Claude 作為後端的深度整合（prompt caching 用 Anthropic API 但行為不同於 Claude Code）

**方案 B：借鑑 Hermes 的技能學習迴圈，整合進現有共腦**

```
現有共腦 + Hermes 式學習迴圈
= 任務完成後自動提取 skill → 受太極約束的技能庫
```

可行性：✅ **高**。不需要引入 Hermes 整個框架，只需：
- 在共腦的 Worker 完成任務後，觸發 skill extraction（Markdown 技能文件）
- 技能庫存在 projects/taichi-framework/skills/ 或類似目錄
- 新技能必須通過「貞」（驗證）才能持久化
- 這完全符合生生循環：感（任務來）→ 化（執行）→ 貞（提取+驗證 skill）→ 新（技能庫更新）

**方案 C：不整合，只借鑑設計理念**

從 Hermes 學到的可直接應用：
- **Frozen Snapshot Pattern**：session 開始時快取 system prompt，保護 prefix cache
- **Periodic Nudge**：固定間隔讓 agent 自省，不依賴使用者觸發
- **字元上限強制**：記憶不無限膨脹，滿時合併
- **安全掃描**：記憶寫入前檢查 injection

---

## 5. 與 Claude Code Agent SDK 的比較

### 5.1 Claude 官方方案

2026-04 Anthropic 推出：
- **Agent Teams**（實驗性）：多個 Claude Code session 協作，一個 team lead 分配任務，teammates 獨立工作+互相溝通
- **Managed Agents**（2026-04-08 公開 beta）：雲端託管 agent 平台，內建沙箱、MCP 整合、多 agent 協調、執行追蹤

### 5.2 核心比較

| 維度 | Claude Code + Agent SDK | Hermes Agent |
|------|------------------------|-------------|
| **模型** | Claude only | 任意 LLM |
| **定位** | 編程 agent + 團隊協作 | 通用個人 agent |
| **記憶** | CLAUDE.md + auto memory | 三層記憶 + 技能庫 |
| **自我改進** | 無內建機制 | 學習迴圈 + 技能提取 |
| **多 agent** | Agent Teams（實驗） + Managed Agents（beta） | 單 agent 為主（#344 issue 討論中） |
| **部署** | Anthropic 雲端 / 本地 CLI | 自託管 |
| **工具生態** | MCP 6000+ 應用 | 40+ 內建 + MCP 互通 |
| **穩定性** | 穩定，企業級 | 早期（v0.7），快速迭代 |
| **成本** | $59-200/月 | $5-50/月（自託管 + API 費） |

### 5.3 對共腦的意義

**Claude Code Agent SDK 更適合共腦的核心**，原因：
1. 共腦 Mono 已經是 Claude Code session，生態無縫
2. Agent Teams 的 team lead + teammates 模式天然對應 Mono + Worker
3. Managed Agents 提供雲端持久化，可能解決 RPi5 算力限制
4. MCP 生態遠大於 Hermes 內建工具

**Hermes 的優勢在 Claude Code 沒有的地方**：
1. 自我改進學習迴圈（Claude Code 沒有）
2. 不鎖定模型（備援場景，呼應 `project_local_distillation.md`）
3. $5 VPS 可跑（成本優勢明顯）

---

## 6. 實作可行性評估

### 6.1 方案推薦：借鑑 > 整合 > 替換

| 優先級 | 方案 | 工作量 | 價值 | 建議 |
|--------|------|--------|------|------|
| 1 | 借鑑設計理念（方案 C） | 低 | 高 | ✅ 立即可做 |
| 2 | 移植學習迴圈到共腦（方案 B） | 中 | 高 | ✅ 驗證週可試 |
| 3 | Hermes 作為備援執行層（方案 A） | 高 | 中 | 🔶 後續評估 |
| 4 | 替換 Claude Code 用 Hermes | 極高 | 低 | ❌ 不建議 |

### 6.2 立即可做的事

1. **Periodic Nudge 機制**：在 Worker 的 watchdog 腳本中加入自省觸發，每 N 個任務讓 agent 回顧產出品質
2. **Skill Extraction**：Worker 完成任務後的 `.worker-result.json` 可擴展為 skill 文件（如果任務夠複雜）
3. **Memory 上限管理**：參考 Hermes 的字元上限 + 合併策略，避免 MEMORY.md 無限膨脹

### 6.3 風險

- Hermes v0.7 仍在早期，API 可能大幅變動
- Nous Research 的長期維護承諾不確定（雖然社群活躍）
- 整合 Hermes + Claude Code 會增加系統複雜度，違反「大部分極保守」原則

---

## 7. 結論

**Hermes Agent 是目前最接近太極框架「生生循環」理念的開源實作**，但它只有「化→貞→新」（執行→驗證→更新），缺少「太極」作為推導錨點。它解決的是「怎麼做」的累積，太極框架解決的是「為什麼這樣做」的治理。

**兩者是互補關係，不是競爭關係。** 最佳策略：借鑑 Hermes 的學習迴圈和記憶管理設計，整合進太極框架作為「器」層的增強，而非引入整個 Hermes 框架。

---

## Sources

- [Hermes Agent GitHub](https://github.com/nousresearch/hermes-agent)
- [Hermes Agent 官方文件](https://hermes-agent.nousresearch.com/)
- [Architecture | Hermes Agent](https://hermes-agent.nousresearch.com/docs/developer-guide/architecture)
- [Persistent Memory | Hermes Agent](https://hermes-agent.nousresearch.com/docs/user-guide/features/memory/)
- [DeepWiki: Memory Systems](https://deepwiki.com/NousResearch/hermes-agent/4.4-memory-systems)
- [How Hermes Agent Memory Actually Works](https://vectorize.io/articles/hermes-agent-memory-explained)
- [Inside Hermes Agent: How a Self-Improving AI Agent Actually Works](https://mranand.substack.com/p/inside-hermes-agent-how-a-self-improving)
- [Claude vs Hermes vs OpenClaw Comparison (Medium)](https://medium.com/@Daniel.O.Ayo/claude-vs-hermes-vs-openclaw-which-ai-agent-is-actually-worth-paying-for-in-2026-81ad77de8225)
- [The New Stack: Persistent AI Agents Compared](https://thenewstack.io/persistent-ai-agents-compared/)
- [Claude Code Agent Teams](https://code.claude.com/docs/en/agent-teams)
- [Claude Managed Agents Guide](https://www.the-ai-corner.com/p/claude-managed-agents-guide-2026)
- [PANews: 47k Stars Analysis](https://www.panewslab.com/en/articles/019d7b37-e1c9-71a8-9af7-74cdf1efa8fd)
- [Hermes Agent Skills Guide](https://hermes-agent.ai/blog/hermes-agent-skills-guide)
- [Multi-Agent Architecture Issue #344](https://github.com/NousResearch/hermes-agent/issues/344)
