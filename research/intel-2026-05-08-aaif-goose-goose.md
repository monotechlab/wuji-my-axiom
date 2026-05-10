# aaif-goose/goose 深度研究 — 對共腦 / 太極框架的 leverage

> Intel 2026-05-08 / 來源：https://github.com/aaif-goose/goose
> 建議動作：對照共腦 extensibility / skill loading 設計

---

## 1. 基本資料 + 事實接地

| 欄位 | 值 | 來源 |
|---|---|---|
| repo full name | `aaif-goose/goose` | gh api repos/aaif-goose/goose（2026-05-08） |
| 描述 | "an open source, extensible AI agent that goes beyond code suggestions — install, execute, edit, and test with any LLM" | 同上 |
| stars | **44,540** | gh api stargazers_count（2026-05-08） |
| forks | **4,555** | gh api forks_count（2026-05-08） |
| open issues | 455 | gh api（2026-05-08） |
| watchers / subscribers | 222 | gh api subscribers_count |
| 主語言 | Rust 48.8% / TypeScript 45.6% | GitHub language stats（2026-05-08） |
| license | **Apache-2.0** | gh api license.spdx_id |
| created_at | 2024-08-23 | gh api |
| 最近 push | 2026-05-08T02:01:54Z（任務派發前 25 分鐘還在動） | gh api pushed_at |
| 最新 release | v1.33.1（2026-04-29） | WebFetch GitHub release |
| topics | acp, ai, ai-agents, mcp | gh api topics |
| 治理歸屬 | 從 `block/goose` 轉到 **Agentic AI Foundation（AAIF）/ Linux Foundation** | README transition note（[未驗證 — WebFetch 摘要，未直接讀 README，但 org name `aaif-goose` 與 topics 一致） |

**性質判定**：production-grade desktop+CLI agent，不是 paper、不是 demo。社群訊號是同類 top-tier（44.5k stars 已和 spec-kit 93k 同數量級，遠高於 ruflo 等）。Apache-2.0 → fork / 借鑑 / 商用都可；治理在 Linux Foundation 旗下 → 中立程度高於單一公司控制（block 原本是 Square 系）。

**核心架構（取自 documentation/docs/goose-architecture/）**：

```
Interface (Desktop / CLI / ACP server)
    │
    ▼
Agent  ── interactive loop ──┐
    │                        │
    ▼                        ▼
Extensions (MCP servers)   Provider (LLM / ACP agent)
    │
    ├── built-in (developer / memory / computer / web …)
    ├── 70+ MCP servers
    └── custom (impl `Extension` trait)
```

**關鍵 trait**（`extensions-design.md` 原文）：

```rust
trait Extension {
    fn name(&self) -> &str;
    fn description(&self) -> &str;
    fn instructions(&self) -> &str;
    fn tools(&self) -> &[Tool];
    async fn status(&self) -> AnyhowResult<HashMap<String, Value>>;
    async fn call_tool(&self, name: &str, params: HashMap<String, Value>) -> ToolResult<Value>;
}
```

四條互補機制：
- **Extensions** ＝ MCP servers，提供 tools / 維護 state
- **Skills** ＝ `~/.agents/skills/<name>/SKILL.md`（YAML frontmatter + body），auto-discover；**明示 backward compat 含 `.claude/skills/`、`.goose/skills/`**；需要 built-in Summon extension（v1.25.0+）
- **Recipes** ＝ YAML，打包 extensions + prompt + settings 成一個可分享 session（含 `/slash-command` 整合與 Recipe Generator URL）
- **Subagents** ＝ delegate to isolated goose instance，5-min timeout；sequential / parallel；可用 recipe 配置（如 `code-reviewer.yaml`）

**Skill 標準對齊**：goose-docs 直接寫 *"goose skills are compatible with Claude Desktop and other agents that support Agent Skills"*，連到 agentskills.io。這是**目前 Claude Desktop / goose / 其他 agent 之間的 skill 事實標準**——和共腦現在借鑑的 addyosmani/agent-skills SKILL.md 是同一套生態。

---

## 2. 對共腦的 leverage（4 個切點）

### 切點 A — spawn worker 編排此技術的工作流？❌ 不適用
共腦 worker = `bash scripts/start-worker.sh`，goose 是同層替代品（CLI agent + extensions）。讓 worker「用 goose」=雙層 agent 套娃，token 加倍、可觀測性下降。否決。

### 切點 B — memory 系統收為 reference？✅ 高優先
新增 `reference_goose_skill_standard.md`：記下 `~/.agents/skills/<name>/SKILL.md` 是 goose+Claude Desktop+其他 agent 的事實標準路徑（含 `.claude/skills/` backward compat），未來共腦自己長 skill 時直接走這個路徑＋格式，自動取得跨工具相容。比單獨借鑑 addyosmani 更精準（addyosmani 也只是這個標準的一個 implementation）。

### 切點 C — daily-intel xray-skills.md 加關鍵字？✅ 中優先
建議加：`agent client protocol`、`acp`、`agentskills.io`、`SKILL.md`、`subagent recipe`、`MCP elicitation`、`mcp-roots`、`mcp-sampling`、`gooseignore`。
理由：goose 把 MCP 規格中還沒被廣泛採用的子協議（elicitation / roots / sampling）都做成獨立 docs 了，這些是未來 12 個月 MCP 生態擴散的領先指標。

### 切點 D — skill metabolism / 共腦治理範式有結構同構借鏡？✅ **最高優先**
goose 把可換組件**正交分為四軸**：

| 軸 | goose | 共腦對照 |
|---|---|---|
| 工具暴露 | Extensions（MCP server）/ tools | scripts/* + worker tool 集合 |
| 知識/流程 | Skills（SKILL.md 自動發現） | 5-substrate metabolism（已有 birth filter / demote） |
| 可分享配置 | Recipes（YAML 打包 extension+prompt） | 焦點 CONTEXT.md + worker prompt（**未打包** — 現在 prompt 散在 issue/dispatch） |
| 任務隔離 | Subagents（一次性、5-min timeout） | tmux worker + .worker-result.json（已有） |

**對共腦的啟示**：第三軸 **Recipes 是共腦目前沒有但缺的拼圖**——派 worker 時的 prompt 模板現在每次都重寫（看 `infra/xray/` 的 dispatch prompt 結構幾乎一樣）。Recipe 本質就是「凍結的 worker spec + 參數化」，能解 lesson-scan + start-worker 之間的接縫。對應 EPOCH-2 階段，是「規則列舉 → 公理推導」之外的第三類抽象：**「重複呼叫的具體腳本 → 帶參的 recipe」**。

> ⚠️ **不要直接 fork goose**：架構正確不代表實作可移植，goose Rust+TS 棧與共腦 bash+python 棧無交集，借鑑概念即可。

---

## 3. 對 Owner 各 project 的 leverage 矩陣

| Project | leverage | 怎麼用 | 理由 |
|---|---|---|---|
| **taichi-framework** | **高** | (a) memory 加 `reference_goose_skill_standard.md`；(b) `templates/SKILL.md` 對齊 `~/.agents/skills/<name>/SKILL.md` 路徑 + YAML frontmatter (name/description)，多賺一個跨工具相容；(c) `templates/RECIPE.md` 新模板探索（worker 任務凍結） | 太極框架本來就要寫 SKILL.md 模板（addyosmani 那條 P0 TODO #59）；goose 是更權威的格式參考 |
| **passive-income** | **高** | 影片量產 pipeline 用 goose recipe 模式凍結（fetch idea → 生腳本 → TTS → 剪輯 → 上架），參數化主題與長度；對齊 `feedback_adobe_automation_real`，把 ExtendScript / aerender 包成 MCP-style tool | recipe 思想直接解「腳本散落」問題；現在 passive-income 的 dispatch 都在重寫流程 |
| **soramimi**（空耳） | **中** | 音頻處理 stage（ASR → 音素對齊 → 音義配對 → 歌詞生成）拆 recipe + subrecipe，對應 goose `subrecipes.md` parallel 模式 | 跨語言空耳是天然的 stage pipeline，subrecipe 結構直接套 |
| **usv** | **中低** | PID Solver 產品化時，把 vehicle spec → optimal PID 凍結成 recipe 給用戶；研究階段不適用（每次參數變動） | `project_usv_pid_product` 是長期路線；研究期不要過早抽象 |
| **omniverse** | **中低** | Isaac Sim 場景生成可包成 recipe（地形/光照/agent 數量參數化）；目前研究階段先不動 | 等 omniverse 進入產品化階段再說 |
| **aquafeed** | **低** | PageIndex 已經卡位了「跨文件查詢」這個槽，goose recipe 不能補；除非 70+ 報告的審查流程要凍結 | 與 PageIndex 重疊不衝突，但增量小 |
| **thesis** | **低** | 文獻調研可用 recipe 凍結「找 paper → 提要 → DOI 查證」流程，但現在 v1.0 已送審、4/30 博班面試已過，邊際效用低 | 時機不對 |
| **cucumber-detection / aidc-aoi / agv** | **低** | 都是已收斂或負結果為主的專案（cucumber 三個負結果記錄在案），不需要新抽象 | — |
| **local-distillation** | **無** | 蒸餾是 model 層問題，goose 是 orchestration 層 | 維度不交 |
| **esp-blast / drink-station / kr6 / pet-id / muse-spark / platycerium** | **無** | 焦點外或硬體導向，與 agent orchestration 無關 | — |

**最高 leverage 兩個**：taichi-framework（範式 + 模板）、passive-income（recipe 凍結量產 pipeline）。

---

## 4. 全面風險評估

| 類型 | 風險 | 嚴重度 | 控制措施 |
|---|---|---|---|
| **技術風險** | (a) goose 是 Rust+TS 重型棧，本機學習成本高；(b) `agentskills.io` 標準可能 6-12 個月後分裂（Anthropic skill 規格與 AAIF 推的不一致） | 中 | (a) **不裝 goose**，只取概念；(b) `templates/SKILL.md` 寫**最大共集 frontmatter**（name+description 為必填，其餘 optional），avoid 鎖死特定方言 |
| **商業授權風險** | Apache-2.0 對借鑑/商用無限制；Apache-2.0 專利條款比 MIT 更保護 fork 者；但 goose **trademark** 仍在 AAIF 手上，公開傳播 Wuji 框架時不可暗示 affiliation | 低 | LANDSCAPE.md 對照段明寫「goose 是同類最大 player，Wuji 不衍生於它」 |
| **路徑依賴鎖定風險** | 太早把 `templates/SKILL.md` 鎖死成 goose 特化路徑（`.agents/skills/`），未來 Anthropic 推新 skill spec 時需要重寫；recipe 概念引入後若不慎，會把共腦從 bash 推向 YAML config-heavy | 中 | (a) 路徑用**雙路徑優先序**：`~/.claude/skills/` ＞ `~/.agents/skills/`（Owner 是 Claude Code 重度使用者，先服務本地）；(b) recipe 先在 passive-income 一個專案試水，**不入太極公版**至少 4 週 |
| **政治與供應鏈風險** | (a) AAIF（Linux Foundation 旗下）vs Anthropic（Skill 標準提案者）的標準戰；block 原本是金融科技公司，敏感領域如果監管壓力轉到 LF 可能影響項目穩定性；(b) repo 在 GitHub（微軟）— 與其他 agent repo 同等風險 | 低-中 | 不依賴 goose 二進位、不嵌入其代碼；只借概念。Skill 標準走「與 Claude Desktop 對齊」優先，agentskills.io 標準是 fallback |
| **社群知識資源風險** | 44.5k stars 看起來健康，但 transition from block 到 AAIF 時間軸不明，**[未驗證]**；治理變更可能讓部分 maintainer 流失，未來 6 個月文檔同步會落後 | 中 | (a) 引用 goose docs 時記時戳（本檔已加 2026-05-08）；(b) 不把 goose 視為長期 reference，6 個月後重新評估 |
| **過度抽象風險（自加項）** | recipe 概念吸引但**現在共腦只有少數 worker 任務在重複**，過早抽象 = 把簡單 bash 變 YAML+模板地獄 | 中 | 至少 ≥3 個重複度高的 worker dispatch（例如 daily-intel 7 段研究模板就是潛在 recipe）才動工，不到門檻不寫 |

---

## 5. 自洽校核（與 Owner memory 對齊）

✅ **對齊**：
- `feedback_use_scripts.md`（重複腳本化）— recipe 概念正是「凍結重複腳本」的延伸
- `project_skill_metabolism.md`（5 層 substrate + birth filter）— goose 四軸正交（extension/skill/recipe/subagent）給 metabolism 補一個 distribution 維度
- `infra_remote_control.md`（RPi5 為 claude.ai 環境）— skill 路徑 `~/.claude/skills/` 現在就在 RPi5 上，goose 對齊這個路徑
- `feedback_focus_not_universe.md` — 本份 leverage 矩陣**主動掃了非焦點專案**（passive-income/soramimi/aquafeed），符合「焦點不是過濾器」原則

⚠️ **注意**：
- `feedback_dont_ask_obvious.md` — 本研究中 transition 時間軸標 [未驗證]，但 stars/forks/license/last push 都查了 gh api，符合「可查的不要問」
- `project_taichi_framework.md` 提到 EPOCH-2 是公理推導，goose 仍是規則列舉層級（recipes/skills 都是顯式規則）— 借鑑 goose 概念不等於太極倒退到規則期，是「器層」工具，不是「道層」推導
- 與已有 addyosmani-agent-skills 調研重疊：addyosmani 是 SKILL.md 9 段 anatomy 的提案者；goose 是 SKILL.md 路徑與 agentskills.io 標準的承載者—**兩者互補不重複**，addyosmani 給內容結構，goose 給檔案系統位置與發現機制

❌ **衝突**：
- `feedback_no_haiku.md` 與本研究無關（不影響）
- `infra_claude_subscription.md`（Max 訂閱）— goose 自己也支援 ACP（用 Claude Code 當 provider），等於「在 Claude 之上再包一層 agent」與 Owner 已用 Claude Code 重複，**不要裝 goose binary**

---

## 6. 立即可動手 actions（低承諾、可逆、不需新硬體 / 不需付費）

1. **[15 min]** 寫 `memory/reference_goose_skill_standard.md`：記下 `~/.agents/skills/<name>/SKILL.md` 與 `~/.claude/skills/` backward compat 是 2026-05 事實標準，附 goose-docs URL + 查證日期。**index 加一行**。
2. **[30 min]** 補 `research/LANDSCAPE.md`（既有 P2 TODO 已有 spec-kit / PageIndex 對照位）：加 goose 段，與 spec-kit 並列為「同類最大 player」，標明 goose=可換器層解、spec-kit=規則治理解、太極=公理推導解，三者作用域不同。
3. **[1 hr]** 草擬 `templates/SKILL.md`（taichi 公版，addyosmani P0 TODO #59 升級版）：採 goose YAML frontmatter（name+description）+ addyosmani Rationalizations/Red Flags/Verification 三段，路徑對照 `~/.claude/skills/` 與 `~/.agents/skills/` 並存。
4. **[2 hr]** 在 `daily-intel.py` 7 段研究模板（剛沿用本研究的結構）抽出 `templates/recipes/intel-research.yaml` 試做 recipe 概念，**只在 daily-intel 一處跑 4 週**，不入公版。
5. **[20 min]** xray-skills.md 加關鍵字：`agentskills.io`、`SKILL.md`、`agent client protocol`、`acp`、`subrecipe`、`mcp-elicitation`、`mcp-roots`、`mcp-sampling`、`gooseignore`，下一輪 daily-intel 自動撒網。

> 全部不需新硬體 / 不需付費 / 不需安裝 goose。

---

## 7. 結論

**判斷：值得投入，但只取概念不取實作。** 切入點集中在兩處：(a) `templates/SKILL.md` 對齊 `~/.agents/skills/` 標準路徑——**這條讓共腦未來長出的 skill 自動跨 Claude Desktop / goose / 其他 agent 相容**，邊際成本接近零、上限大；(b) Recipe 概念給共腦 worker dispatch 補一個「凍結重複 prompt」的中間層，但**先試 1 個專案（daily-intel）4 週**驗證重複度真到門檻才入公版。

**But-For 推導**：若不做這份研究，共腦的 SKILL.md 模板會純跟 addyosmani（內容對但路徑無標準），等於主動退出 agentskills.io 跨工具相容；若做了，多賺一個生態相容維度，且未來 Anthropic 若推官方 skill spec，本檔已記下回退路徑。Recipe 概念若不入觀測，6 個月後共腦 worker dispatch 仍會散落在 issue/script/prompt 三處——這是 goose 已踩過的坑，借鑑是 strict win。

**唯一風險**：路徑鎖定（過早寫死 `.agents/skills/` 而 Anthropic 後出新 spec）。控制方法：模板雙路徑並存，主路徑優先 `~/.claude/skills/`，goose 路徑作 backward compat。

**整體 leverage 高於 spec-kit 調研**（spec-kit 仍是規則列舉，goose 是可換器層 + 跨工具相容標準）；**低於 addyosmani**（addyosmani 直接給內容模板，goose 給生態位）；**互補不替代**。
