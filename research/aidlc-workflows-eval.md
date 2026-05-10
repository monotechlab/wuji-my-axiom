# aidlc-workflows 深度評估

> 調研日期：2026-05-08
> 對象：[awslabs/aidlc-workflows](https://github.com/awslabs/aidlc-workflows)
> Clone snapshot：default branch `main`（pushed_at 2026-05-05T17:19Z）

## 1. 基本資料

| 項目 | 值 | 來源 |
|------|----|------|
| Owner | `awslabs`（AWS 官方 organization） | gh api repos/awslabs/aidlc-workflows |
| 性質 | "AI-Driven Life Cycle (AI-DLC) adaptive workflow steering rules for AI coding agents" | repo description |
| Stars / Forks / Open Issues | 1585 / 299 / 44 | gh api（2026-05-08 13:30Z） |
| Created / Last push | 2025-11-13 / 2026-05-05 | gh api |
| License | **MIT-0**（MIT No Attribution，可自由 fork/改/閉源不需署名） | gh api `license.spdx_id` |
| 主語言 | Python（實際 99% 是 Markdown 規則 + Python 評估器） | gh api `language` |
| 活躍度 | 6 個月內 1.5k stars + 299 forks，44 個 open issue → 高熱度，仍在快速演化 | gh api |
| 核心交付 | `aidlc-rules/` zip 包，掛 GitHub Releases | AGENTS.md L13 |

不是 LangGraph 那種 Python 框架，**是一包 Markdown 規則 + 評估器**，掛進 IDE-native agent（Kiro / Amazon Q / Cursor / Cline / Claude Code / GitHub Copilot / OpenAI Codex）當 steering files / system prompt 用。

---

## 2. 核心功能

### 2.1 全名解碼

**AI-DLC = AI-Driven Development Life Cycle**（README.md L1）。Amazon 自家 method paper（aws.amazon.com/blogs/devops/ai-driven-development-life-cycle/）背書的「給 LLM 用」的 SDLC。

### 2.2 三階段適應性 workflow

從 `aidlc-rules/aws-aidlc-rules/core-workflow.md` 提取（總共 539 行，是整個系統的入口）：

```
INCEPTION    → Workspace Detection (ALWAYS) → Reverse Engineering (CONDITIONAL)
                → Requirements Analysis (ALWAYS, adaptive depth)
                → User Stories (CONDITIONAL) → Workflow Planning (ALWAYS)
                → Application Design (CONDITIONAL) → Units Generation (CONDITIONAL)
CONSTRUCTION → 對每個 unit 跑：Functional Design / NFR Req / NFR Design / Infra Design
                → Code Generation (ALWAYS, per-unit) → Build & Test (ALWAYS)
OPERATIONS   → 留白 placeholder
```

13 個 stage，每個都標 **ALWAYS / CONDITIONAL**。LLM 自己根據三個輸入判斷該跑哪些（core-workflow.md L4-11）：用戶意圖清晰度 / 既存 codebase 狀態 / 複雜度與風險。每個 stage 結束**強制等用戶批准**才推進（重複出現 `Wait for Explicit Approval` 與 `DO NOT PROCEED until user confirms`，core-workflow.md L136、L157、L226 等）。

### 2.3 Adaptive depth

每個 stage 三檔：minimal / standard / comprehensive（core-workflow.md L141-145）。LLM 判斷複雜度自動選。

### 2.4 強制 audit log

`audit.md` 是 single source of truth（core-workflow.md L455-507）：
- ISO 8601 timestamp
- **完整原始 user input**，禁止摘要（"NEVER summarize or paraphrase user input in audit log"）
- 每個 approval / 每個 user response 都要記
- 工具上：append-only，**禁止整個檔案 rewrite**（會造成重複）

### 2.5 Extension opt-in 機制（context 優化）

extensions/ 目錄裡每個擴展兩份檔案（core-workflow.md L29-50）：
- `<name>.opt-in.md`：輕量 prompt，工作流啟動時**全部載入**
- `<name>.md`：完整規則，**只在用戶 opt-in 後 lazy load**

例：`extensions/security/baseline/security-baseline.opt-in.md` 先載，用戶選了才載 `security-baseline.md`。Context 大幅省，目前看到 testing/property-based + security/baseline 兩個範例擴展。

### 2.6 評估器（scripts/aidlc-evaluator/）

獨立的 Python uv workspace，**8 個 package** 架構（ARCHITECTURE.md L41-50）：

| Package | 職責 |
|---------|------|
| execution (`aidlc-runner`) | 跑 **2-agent Strands Swarm**（Executor + Human Simulator）執行整套 AIDLC workflow |
| qualitative | LLM 對比生成 docs 與 golden baseline |
| quantitative | 靜態分析（lint / 安全掃描 / 重複度） |
| contracttest | 生成 app 起來打 OpenAPI 契約測試 |
| nonfunctional | NFR 評估（token 消耗 / 計時 / 跨模型一致性） |
| reporting | 合併產 report.md / report.html |
| ide-harness | 包裝第三方 IDE adapter |
| shared | 公用 |

關鍵設計（ARCHITECTURE.md L74-78）：**所有 package 透過 disk YAML 通訊**，不在 in-process 互相 import；orchestrator 用 `python -m <pkg>` subprocess 跑。每個 package 獨立可測。

Run folder 慣例（runner.py L47-77）：`runs/<ISO8601>-<rules-slug>/`，含 sentinel `.last_run_folder` 指向最新 run（atomic via `os.replace`），父 orchestrator 不需 racy diff 列目錄。

### 2.7 與 LangGraph / AutoGen / CrewAI 比較

| 維度 | aidlc-workflows | LangGraph | AutoGen | CrewAI |
|------|-----------------|-----------|---------|--------|
| 範式 | **規則包** (Markdown steering files) | 圖式狀態機 | 多 agent 對話 | role-based crew |
| 執行載體 | IDE 內建 LLM agent（不自帶 runtime） | Python 進程 | Python 進程 | Python 進程 |
| 控制流位置 | LLM 自己讀規則決定 | 程式碼顯式 graph edges | message routing | task delegation |
| User approval gate | **強制每階段** | 可選（human-in-loop node） | 可選 | 可選 |
| Audit | **強制 append-only audit.md** | LangSmith trace | log file | 自訂 |
| Multi-agent | **沒有**（只有 evaluator 內 2-agent swarm 用 Strands） | 是 | 是 | 是 |
| 套件數 | 0（純 markdown）；evaluator 8 包 | 多 | 多 | 多 |

**結論：aidlc-workflows ≠ agent orchestration framework**，更像是「給 IDE-LLM 看的 SDLC 教科書 + 流程紀律」。最近的類比是 Claude Code 的 Skills / Kiro 的 Steering Files。它把「治理協議」推到 LLM 推理層，不在 host-side 加 runtime。

---

## 3. 對齊判斷：對共腦 worker 編排有無借鑑

對 `scripts/start-worker.sh` + `scripts/brain-watchdog.sh` 來說，**架構方向不同**，但有具體可借的零件：

### 3.1 不直接借鑑的地方

- **強制 user approval** vs 共腦 `feedback_full_autonomy.md` / `feedback_dont_ask_obvious.md`：方向相反，共腦明確要 worker 不問就做。AI-DLC 的 13 個 `Wait for Explicit Approval` 套到共腦反而違反爻則。
- **13-stage SDLC** 對研究/原型/單發 worker（如 cucumber 標注、USV PID 調參）過重。共腦目前 worker 是「一發任務一個 .worker-result.json」，套 SDLC 是 over-engineering。
- **Bedrock-Strands 綁定**：evaluator 寫死 AWS Bedrock + Strands SDK（executor.py L1-15），對共腦不可移植。

### 3.2 可借的零件（按優先序）

**A. Audit log schema 升級 `.worker-result.json` → `.worker-result/audit.md`**
- 現狀：worker 跑完只寫一份 `.worker-result.json`，事故時無法重建決策軌跡。
- AI-DLC 提供成熟 schema（core-workflow.md L487-495）：每個 user input 完整原文 + AI action + stage context + ISO timestamp，append-only。
- **對共腦最大好處**：post-mortem 時可看到 worker 中途的 self-correction 與被 watchdog 中斷的時點。

**B. Two-Level Checkbox 模式（core-workflow.md L463-474）**
- Plan-level（worker 內任務 step）+ Stage-level（aidlc-state.md，跨 stage 進度）
- 共腦目前 TODO.md 是 stage-level，但 worker 內的 sub-step 沒有顯式追蹤；對應重型 worker（如 thesis pipeline）可能有用。
- 注意：「IMMEDIATELY mark [x] in same interaction」是強制 LLM 不忘記更新進度的具體機制，可抄。

**C. Adaptive depth flag**
- start-worker.sh 派工 prompt 加 `--depth=minimal|standard|comprehensive` 參數，把 AI-DLC 的判準（user intent clarity / existing state / complexity / risk）內化進去。
- 對齊 ruflo 調研那條 P1 「Queen 類型語意化（strategic/tactical/adaptive mode flag）」。

**D. Overconfidence-prevention（overconfidence-prevention.md）**
- AWS 獨立寫了一份「default to asking, when in doubt ask」的反過度自信指南，與共腦 `feedback_overconfidence_signal.md` 同方向。
- **不需借**，但這是業界第三方驗證共腦該方向是對的（continuing 的 lesson confirmation）。

---

## 4. 對 daily-intel.py 多 source pipeline 的啟發

這條是**最直接可動手**的方向。AI-DLC evaluator 的 6-stage pipeline 與 daily-intel 的「多 source → 抓取 → 排重 → 寫入 inbox」結構同構，但工程紀律高一截。

### 4.1 借鑑 1：subprocess + YAML on disk 的 stage 解耦

**現狀（推測）**：daily-intel.py 是單體腳本，所有 source 處理在同一 process。
**AI-DLC 模式**（ARCHITECTURE.md L74-78）：
- 每 stage 一個 package，orchestrator 用 `python -m <pkg>` subprocess 起
- stage 之間靠 YAML 檔通訊，不靠 in-process import
- 結果：單 stage 可獨立 debug / 重跑 / 跨機跑

對 daily-intel：可以拆 `fetchers/<source>` (孤立 fetch) → `dedupe/` (跨 source 排重) → `classify/` (D/X/W tag) → `inbox/` (寫到 TODO)，stage 間用 YAML。**最大好處**：某 source 掛掉只需重跑那一 stage。

### 4.2 借鑑 2：run-folder + sentinel 慣例

`runs/<ISO8601>-<slug>/` + `.last_run_folder` sentinel（runner.py L44-75）：
- 父 orchestrator 不靠 `ls runs/` diff 找最新（race-prone），讀 sentinel
- atomic 寫入（os.replace）
- 失敗的 run 也保留完整現場可回查

對 daily-intel：每天的 intel run 起一個 timestamped folder，所有 source 的 raw + 處理中間檔都落地，事故可重建。

### 4.3 借鑑 3：Strict YAML 中間檔 schema

evaluator 每個 stage 寫固定 schema YAML（README.md L60-72）：`run-meta.yaml` / `run-metrics.yaml` / `test-results.yaml` / `quality-report.yaml`...

對 daily-intel：每個 source 的 fetch 結果落 `<source>-fetch.yaml`（含 timestamp / 抓到幾筆 / latency / 錯誤），dedupe 落 `dedupe-stats.yaml`。比現在 `infra/github-trending.json` 單檔 mutate 更可觀測。

### 4.4 不借鑑的

- 2-agent Strands Swarm（executor.py + simulator.py）：對 daily-intel 過於 heavyweight，daily-intel 不需要 LLM-in-loop。
- Bedrock 綁定：共腦走 Claude Code，不引 boto3。

---

## 5. 立即可動手的 Next 3 Actions

依 ROI / 可逆性排序：

### Action 1（P0，最高 ROI）— Worker audit.md schema 升級

**做什麼**：把 `scripts/start-worker.sh` 派 worker 時注入的 prompt 模板加一條：
> 工作期間每個關鍵決策、每次工具呼叫前後、每次方向變更，必須 append 到 `.worker-result/audit.md`，schema 嚴守 ISO timestamp + 原文 input + action + context（參考 core-workflow.md L487-495）。

並在 worker 退出協議中要求把 `.worker-result/audit.md` 與 `.worker-result.json` 一併 commit。

**為什麼**：當 worker 跑歪（如「燒韌體前發現 haiku 編造引腳」、「過期數據當現狀」），目前只有最終 result.json 看不到中途。這條直接強化 `feedback_trust_breach` + `feedback_factcheck_before_deliver` 的可驗證性。

**可逆性**：完全可逆，純 prompt 模板改動，worker 不寫也只損失 audit。

---

### Action 2（P1）— daily-intel.py 拆 stage + YAML on disk

**做什麼**：
1. 拆 `daily-intel.py` 成 `fetchers/<source>.py` + `dedupe.py` + `classify.py` + `inbox.py`，orchestrator 用 `python -m` subprocess 串
2. 每 stage 寫 timestamped run folder（`runs/daily-intel/<ISO8601>/`），中間 YAML 落地
3. orchestrator 寫 `.last_run_folder` sentinel 供 watchdog 讀

**為什麼**：對齊「持續研究與自省」(`feedback_continuous_research`)，daily-intel 是共腦感知外界的主動脈。目前單體腳本失敗難 diagnose，拆完後 source-level 韌性大增。

**可逆性**：中等。需要重寫 daily-intel.py 主流程，但邏輯不變只是切割，可在 branch 上開發後 swap。

**前提檢查**：先 `Read /home/ra/ai-group/scripts/daily-intel.py` 確認當前實際結構（CONTEXT.md 的描述未對 ground truth 比對，**未驗證**）— 落實 `feedback_verify_premise_before_proposing`。

---

### Action 3（P2）— Lesson 系統 lazy-load 機制

**做什麼**：把 AI-DLC `<name>.opt-in.md` + `<name>.md` 雙檔模式套到 `scripts/lesson-scan.sh`：
- `lessons/<topic>.preface.md`：30 行內的 trigger description + 何時相關，**永遠載入**到 worker prompt
- `lessons/<topic>.full.md`：完整 lesson body，worker 透過工具呼叫（如 `lesson-load <topic>`）按需取

**為什麼**：lesson 庫長大後 worker 啟動 token 成本線性增長（已被 `feedback_use_scripts` 壓 token 的方向）。AI-DLC 的 opt-in 模式（core-workflow.md L29-50）證明這是 production-grade 解。

**可逆性**：中等偏可逆。需要拆現有 lesson 檔成兩份，但格式可工具化批次轉。

---

## 附錄：file:line 引用清單

- repo metadata：`gh api repos/awslabs/aidlc-workflows`
- 三階段 + adaptive：`aidlc-rules/aws-aidlc-rules/core-workflow.md:1-110`
- approval gates：`core-workflow.md:136,157,226,266,331,387,420`
- audit schema：`core-workflow.md:455-507`
- extension opt-in：`core-workflow.md:29-50`
- two-level checkbox：`core-workflow.md:463-474`
- evaluator 6-stage：`scripts/aidlc-evaluator/README.md:30-50`
- 8-package 架構：`scripts/aidlc-evaluator/ARCHITECTURE.md:41-78`
- run folder + sentinel：`scripts/aidlc-evaluator/packages/execution/src/aidlc_runner/runner.py:44-77`
- 2-agent swarm：`packages/execution/src/aidlc_runner/agents/executor.py:18-29`、`simulator.py:16-78`
- overconfidence prevention：`aidlc-rules/aws-aidlc-rule-details/common/overconfidence-prevention.md`
- session continuity：`aidlc-rules/aws-aidlc-rule-details/common/session-continuity.md`
- process overview + Mermaid：`aidlc-rules/aws-aidlc-rule-details/common/process-overview.md`
