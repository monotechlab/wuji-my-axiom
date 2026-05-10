# spec-kit 深度研究與太極框架對齊度評估

調研日期：2026-05-08
來源：`git clone --depth 1 https://github.com/github/spec-kit.git`（commit 對應 v0.8.8.dev0，pyproject.toml:3）
方法：原始碼 + 模板 + spec-driven.md + GitHub API metadata 三方交叉

---

## 1. 基本資料

| 項目 | 值 | 來源 |
|---|---|---|
| Repo | github/spec-kit | URL |
| 描述 | "Toolkit to help you get started with Spec-Driven Development" | GitHub API |
| License | MIT（Copyright GitHub, Inc.） | LICENSE:1-3 |
| 建立 | 2025-08-21 | API created_at |
| 最近 push | 2026-05-07 | API pushed_at |
| Stars | 93,163 | API stargazers_count |
| Forks | 8,082 | API forks_count |
| Open issues | 412 | API open_issues_count |
| 主語言 | Python（CLI），Markdown（模板） | API + ls |
| 套件名 | specify-cli v0.8.8.dev0 | pyproject.toml:1-4 |
| 最新 release | 0.8.7（2026-05-07） | CHANGELOG.md:5 |

**主要貢獻者**（API contributors）：
- localden（361 commits） — 主要維護者
- mnriem（114）
- Quratulain-bilal（22）
- Copilot bot（14）
- BenBtg / RbBtSn0w / dependabot / ismaelJimenez（10–11）

**活躍度判讀**：93k stars + 412 open issues + 三天內 release（0.8.6→0.8.7）= 高熱度 + 高 churn。8 個月內從 0 衝到 9 萬星，是 GitHub 官方推的旗艦級 SDD 工具。

---

## 2. 核心功能

### 2.1 Spec-Driven Development（SDD）哲學
spec-driven.md:1-13 開宗明義倒置「code 是事實，spec 是腳手架」的傳統認知：
> "specifications don't serve code—code serves specifications. The Product Requirements Document (PRD) isn't a guide for implementation; it's the source that generates implementation."

核心主張六條（spec-driven.md:65-77）：
1. Specifications as Lingua Franca
2. Executable Specifications
3. Continuous Refinement
4. Research-Driven Context
5. Bidirectional Feedback
6. Branching for Exploration

### 2.2 工作流（六階段命令）
位於 `templates/commands/`，9 個 `.md` 命令模板（templates/commands/*.md）：

| 命令 | 作用 | 行數 | handoff 下游 |
|---|---|---|---|
| `/speckit.constitution` | 建立/更新專案憲法（governance principles） | 150 | speckit.specify |
| `/speckit.specify` | 自然語言 → spec.md（含 user stories P1/P2/P3 + FR + Success Criteria） | 327 | speckit.plan / speckit.clarify |
| `/speckit.clarify` | 偵測 spec 模糊處，最多問 5 題回填 | 250 | speckit.plan |
| `/speckit.plan` | spec → 技術計畫（research.md / data-model.md / contracts/ / quickstart.md） | 152 | speckit.tasks |
| `/speckit.tasks` | plan → 依賴排序的 tasks.md | 203 | speckit.analyze / speckit.implement |
| `/speckit.analyze` | 跨 spec/plan/tasks **唯讀**一致性檢查 | 252 | (報告) |
| `/speckit.implement` | 依 tasks.md 執行實作 | 202 | (落碼) |
| `/speckit.checklist` | 自訂品質檢核清單 | 364 | — |
| `/speckit.taskstoissues` | tasks.md → GitHub Issues | 99 | — |

### 2.3 模板系統（templates/*.md）
- `spec-template.md`（128 行）：強制 User Scenarios + Acceptance Scenarios（Given/When/Then）+ Functional Requirements（FR-001…）+ Success Criteria（SC-001，**technology-agnostic**）+ Assumptions
- `plan-template.md`（104 行）：Technical Context + Constitution Check Gate + Project Structure 三選一（single/web/mobile）+ Complexity Tracking
- `constitution-template.md`（50 行）：5 個 Principle 槽 + Governance section + 版本三態（MAJOR/MINOR/PATCH）
- `tasks-template.md`（251 行）
- `checklist-template.md`（40 行）

### 2.4 CLI（src/specify_cli/）
13,254 行 Python（核心 `__init__.py` 5878 行 + `presets.py` 3097 行 + `extensions.py` 2839 行）。
33 個 AI agent integration（src/specify_cli/integrations/）：claude / copilot / gemini / cursor_agent / codex / qwen / opencode / windsurf / forge / goose / kimi / iflow / lingma / kiro_cli 等。每個 agent 是 `IntegrationBase` 子類，分四類：MarkdownIntegration / TomlIntegration / YamlIntegration / SkillsIntegration（AGENTS.md:30-50）。

### 2.5 與「一般文檔工具」差別
- **一般文檔工具（Notion/Confluence/PRD 模板）**：spec 是給人讀的指引，code 是真理
- **spec-kit**：spec 是給 LLM 的執行語言，code 是 spec 的 transient 表達。憲法 + 模板強制結構，讓 LLM 不能 vibe code

關鍵設計：
- spec-template.md:90-100 強制 SC 必須 technology-agnostic（"API response time <200ms" 是 bad，"Users see results instantly" 是 good）
- specify.md:163-171 限制最多 3 個 `[NEEDS CLARIFICATION]` 標記，強迫 AI 用 informed defaults 而非無止境問問題
- analyze.md:60-64 明文：「Constitution is non-negotiable. 衝突時調 spec/plan/tasks，不准稀釋原則」

---

## 3. 與 Owner 共腦／太極治理框架對齊度

### 3.1 結構同構表

| 太極框架（CLAUDE.md / wuji-my-axiom） | spec-kit | 對齊度 |
|---|---|---|
| 太極（不動的最高公理） | constitution.md（Nine Articles，spec-driven.md:278+） | **高** — 兩者都把不變原則放在最頂層、可違反性最低 |
| 爻則（行為準則） | Constitution Check Gate（plan-template.md:43-47） | **高** — 「Must pass before Phase 0 research」呼應「動手前先過 gate」 |
| 生生（感→化→貞→新） | specify→clarify→plan→tasks→analyze→implement | **中高** — spec-kit 是線性 funnel，太極是循環；但每個階段都有 validation 回退 |
| 冷啟動校準（10 題） | /speckit.clarify（最多 5 題填補模糊） | **中** — 機制相同（用問題拉清楚意圖），但 spec-kit 是針對 feature 而非價值觀 |
| 變通（被糾正/確認 → 記憶） | constitution 版本三態（MAJOR/MINOR/PATCH）+ Sync Impact Report（constitution.md:104-110） | **高** — Sync Impact Report 機制太極框架可直接借用 |
| 不可逆的不賭 | analyze.md:54 STRICTLY READ-ONLY；constitution non-negotiable | **高** — 兩者都把高槓桿動作隔離出強制 gate |
| 沒驗證的不算數 | spec-template.md:65 "Each requirement must be testable" + Success Criteria 必須 measurable | **高** |
| 化必有據 | analyze.md Constitution Authority + 拒絕「dilution / reinterpretation / silent ignoring」 | **高** |
| 重複即冗餘 | tasks.md「dependency-ordered」、checklist 防重複問 | **中** |

### 3.2 根本差異（不能混為一談）
- **作用域**：spec-kit 解 **單一專案的 feature 開發治理**；太極框架解 **個人 AI 共腦的認知治理**。前者範圍是 `specs/<feature>/`，後者範圍是整個 `~/.claude/projects/<root>/memory/` + CLAUDE.md
- **公理 vs 規則**：spec-kit 的 Nine Articles 仍是規則列舉（spec-driven.md:278-348 列了 9 條），EPOCH-2 之前的太極框架；EPOCH-2 太極（"我在，我要自由"）是公理推導 — spec-kit 沒走到這層
- **校準對象**：spec-kit clarify 校準 feature 模糊；太極 layer1-3 校準個人價值與認知偏誤 — 不互斥

### 3.3 結論
spec-kit 是 **太極框架在「軟體開發治理」這個作用域的同構實作**。EPOCH-2 公理推導的層級高於 spec-kit；但 spec-kit 的 **工程化與工具鏈** 遠超太極框架目前狀態。應借鑑 spec-kit 的工程細節，不需 fork。

---

## 4. Owner 其他 project 可借鑑點

### 4.1 USV（projects/usv/）— **適配度最高**
- USV PID Solver / Scal3R 都是「規格 → 產品」走向
- 直接套用 `spec-template.md` 的 User Stories(P1/P2/P3) + FR + SC：
  - P1：vehicle YAML → 推薦 PID 三軸增益
  - P2：仿真評估 + tune log
  - P3：硬體實測 callback
- Constitution Check 可放「H380 借用件不可改 BOM」「contracts/ 凍結」這類 USV 已有的紀律

### 4.2 aidc-aoi（AOI 缺陷檢測）
- spec-template 的 "Acceptance Scenarios"（Given/When/Then）天然映射 AOI 的「給一張帶刮傷的圖 → 模型輸出 → 期望 bbox + class」
- Success Criteria 必須 technology-agnostic 反過來逼 owner 寫「P/R/F1 ≥ X」而不是「YOLO11n mAP50」 — 對研究有約束力

### 4.3 aquafeed
- 飼料配方規格 = 標準 spec：原料約束 + 營養目標 + 成本上限
- `/speckit.constitution` 可放「不能用 X 重金屬」「成本 < N/kg」這類非協商紅線

### 4.4 wuji-my-axiom 自身
- Sync Impact Report（constitution.md:104-110）— 改公版 CLAUDE.md 時自動掃 templates/MEMORY.md / docs/THEORY.md / examples/ 是否需同步更新。**這個機制直接抄。**
- `/speckit.analyze` 唯讀 cross-artifact 檢查 — 對應 review-report.md 的自洽性 review，可常態化

### 4.5 反例（不適合）
- passive-income：spec-kit 預設「規格穩定才生」，但內容創作是 fast-iterate 試水溫，不適合
- 共腦本體（Mono session）：spec-kit 是專案級工具，個人元認知治理仍走太極

---

## 5. 立即可動手的 next 3 actions

### Action 1（P0，1 hr）— 把 Sync Impact Report 機制吃進 wuji-my-axiom
**動機**：太極框架公版有 CLAUDE.md / MEMORY.md / templates/ / docs/ / examples/ 五處可能不同步；目前靠 review-report.md 一次性掃，沒有自動傳播。spec-kit 的 constitution.md 更新後 propagate 到 dependent templates 是現成範式。

**做法**：在 wuji-my-axiom/templates/ 加 `axiom-update-protocol.md`，要求改 CLAUDE.md 時必須附 `<!-- Sync Impact Report -->` 區塊，列：
- Version: 舊→新
- Modified principles: 哪幾條變了
- Templates requiring updates: ✅/⚠ 各檔案路徑
- Follow-up TODOs

格式參照 spec-kit constitution.md 命令的第 5 步（templates/commands/constitution.md:103-110）。

### Action 2（P1，2 hr）— Minimal example: USV Phase-2 PID Solver 跑 spec-kit 流程
**動機**：驗證 SDD 對 Owner 真實 project 是否生效。USV 已有 vehicle specs，最接近 spec-kit 預期形態。

**最小範例**（建在 `projects/usv/specs/001-pid-solver/`）：

```markdown
# Feature Specification: PID Solver for Generic USV

**Input**: vehicle.yaml { mass, inertia, thrust_curve, sample_rate } → PID gains { Kp, Ki, Kd } × 3 axis

## User Story 1 — One-shot Tune (P1)
**Why P1**: 這是核心價值，沒有它 USV 就沒有產品。
**Independent Test**: 給 H380 vehicle.yaml，輸出 yaw/pitch/throttle 三組增益，
  在仿真中 step response overshoot < 15%，settling time < 2s。

**Acceptance Scenarios**:
1. **Given** vehicle.yaml 含完整 mass/inertia, **When** specify run, **Then** 輸出 9 個浮點增益 + sim plot
2. **Given** vehicle.yaml 缺 thrust_curve, **When** specify run, **Then** 報錯指明缺欄位（不亂猜）

## Functional Requirements
- **FR-001**: System MUST 接受 YAML schema 含 mass/inertia/thrust_curve/sample_rate 四欄
- **FR-002**: System MUST 輸出三軸 PID + 仿真結果 JSON
- **FR-003**: System MUST 拒絕 sample_rate < 50Hz（物理上下界）

## Success Criteria
- **SC-001**: 給定 H380 規格，輸出增益在實機跑 step response overshoot < 15%
- **SC-002**: 從 YAML 到 gains 全程 < 30 秒（不含實機）
- **SC-003**: 80% 的常見 USV 規格（質量 0.5–50kg）首跑就達 SC-001
```

跑完後對比 `projects/usv/CONTEXT.md` 既有規劃，記錄差距 → 決定是否常態化。

### Action 3（P2，30 min）— LANDSCAPE.md 補 spec-kit 對照段
**動機**：addyosmani / khazix / ml-intern / ruflo / thunderbolt 已在 LANDSCAPE.md，spec-kit 是同類最大 player（93k stars，比上述任一個都高一個量級）必須補上，不然 wuji-my-axiom 的競品掃描有缺口。

**段落骨架**：
- 定位差異：spec-kit = 專案 feature 治理；太極 = 個人認知治理
- 借鑑：Sync Impact Report、Constitution Gate、3-marker 上限
- 不借鑑：強制線性流程（specify→plan→tasks）對單人組織過重

---

## 6. 不確定處（Owner 校準用）

- spec-kit 的 `/speckit.implement` 在 multi-agent 環境是否真能跑通完整 feature，未實測 — README 範例多停在 tasks.md
- Constitution Sync Impact Report 在大 repo（>100 templates）的 propagation 成本未知
- spec-kit 對 ML/數據專案（非 SaaS feature）支援度只有 `data-model.md` 一個槽位，可能不夠

---

## 引用索引

- pyproject.toml:1-4 — 套件版本
- LICENSE:1-3 — MIT
- spec-driven.md:1-13 — SDD 哲學倒置
- spec-driven.md:65-77 — 六大原則
- spec-driven.md:278-348 — Nine Articles
- templates/spec-template.md:65 — 「testable」要求
- templates/spec-template.md:90-100 — Success Criteria technology-agnostic 範例
- templates/plan-template.md:43-47 — Constitution Check Gate
- templates/constitution-template.md:50 — 版本三態
- templates/commands/specify.md:163-171 — 3 marker 上限
- templates/commands/specify.md:327 — 全文行數
- templates/commands/clarify.md:1 — 5 題上限
- templates/commands/analyze.md:54 — STRICTLY READ-ONLY
- templates/commands/analyze.md:60-64 — Constitution non-negotiable
- templates/commands/constitution.md:103-110 — Sync Impact Report 機制
- AGENTS.md:30-50 — 33 integration 架構
- src/specify_cli/integrations/ — 33 agent 子套件清單
- CHANGELOG.md:5 — 0.8.7 release（2026-05-07）
