# Intel 2026-05-08: vercel-labs/open-agents

> Cloud agent template 比對共腦 worker dispatch 架構

來源：https://github.com/vercel-labs/open-agents
查證日期：2026-05-08

---

## 1. 基本資料 + 事實接地

| 項目 | 值 | 來源 |
|---|---|---|
| 全名 | vercel-labs/open-agents | gh API |
| 描述 | "An open source template for building cloud agents" / "open-source reference app for building and running background coding agents on Vercel" | gh API + README |
| License | MIT | gh API `licenseInfo.key=mit` |
| 主語言 | TypeScript | gh API |
| Stars | 5,067 | gh API 2026-05-08 |
| Forks | 636 | gh API 2026-05-08 |
| 建立日期 | 2025-12-26 | gh API |
| 最近 push | 2026-05-06 08:41 UTC | gh API（活躍，2 天前） |
| 近 5 commit 主題 | 安全強化（DeepSec findings ×2）、session botid 修補、refresh-token script、docs | gh API commits |
| Archive 狀態 | false（活躍） | gh API |

**性質判定：reference app / template** — README 自陳「meant to be forked and adapted, not treated as a black box」。非 framework / 非 SaaS / 非 library，是「Vercel 雲端 agent 部署的官方 reference implementation」。

**核心架構（README 直引）**：「Web -> Agent workflow -> Sandbox VM」，「The agent does not run inside the VM. It runs outside the sandbox and interacts with it through tools like file reads, edits, search, and shell commands.」

**關鍵依賴**：
- Vercel Workflow SDK（durable multi-step execution）
- Vercel Sandbox（snapshot-based VM）
- Next.js + Bun + TypeScript
- PostgreSQL via Neon（每 preview 自動 fork DB branch）
- Better Auth（GitHub App + Vercel OAuth）
- ElevenLabs voice transcription（optional）

**Skills 系統**：`.agents/skills/` 13 個 skill（agent-browser / ai-sdk / baseline-ui / chat-sdk / code-review / deploy-open-harness / emil-design-eng / frontend-design / plan-mode / remove-demo-limits / vercel-react-best-practices / web-animation-design / workflow）+ `skills-lock.json`（skill loader 機制）。

**[未驗證]** subagents 具體定義 / skill 內部 anatomy（README 提及 `packages/agent` 含「agent implementation, tools, subagents, skills」但未深入讀檔）。

---

## 2. 對共腦的 leverage（4 切點判斷）

### 切點 A：spawn worker 編排此技術的工作流？❌ 不適用
共腦 worker = RPi5 tmux + claude CLI，不部署到 Vercel；open-agents 是 web-first cloud-native，與「Owner 在哪共腦就在哪」哲學正交。

### 切點 B：memory 系統收為 reference？✅ 適用（高）
**理由**：open-agents 是「agent-outside-sandbox」這個分離模式的官方參考實作。共腦目前 worker 與 host 共享檔案系統（同一 git 倉），此架構的 isolation 邊界值得對照思考——尤其當未來要外接「不受信工作流」（如客戶交付的腳本）時。

**memory action**：補一條 `reference_vercel_open_agents.md` — 記錄「durable workflow + ephemeral sandbox + agent host 三分」是 cloud-agent 業界 2026-Q2 reference，並記下與共腦「git-as-truth + tmux-as-runtime」的對應關係。

### 切點 C：daily-intel xray-skills.md 加關鍵字？✅ 適用（中）
建議加：`durable workflow agent` / `agent sandbox isolation` / `vercel sandbox snapshot` / `skill-lock`（後者可掃 skills 分發/版本鎖機制這條線，與 xingkongliang/skills-manager 同主題）。

### 切點 D：skill metabolism / 治理結構同構借鏡？✅ 適用（中）
**結構同構點**：
- 13 個 skill 已分類為「framework knowledge（ai-sdk/chat-sdk/workflow）/ design（baseline-ui/frontend-design）/ workflow modes（plan-mode/code-review）/ deploy（deploy-open-harness）」——共腦 5-substrate metabolism 可對照其 de facto 分類驗證 birth filter 條件
- `skills-lock.json` 是 skill artifact 版本鎖，類似 npm lockfile——共腦 skill metabolism 目前無 lock，這是潛在補強點
- `plan-mode` skill 與共腦「先 plan 再做」有思想同構

**反向收穫**：open-agents 13 skill **無 lifecycle / 無 demote / 無 birth filter**（目視 directory listing），與 addyosmani/agent-skills 同病。再次佐證「共腦 metabolism 在 lifecycle 維度領先業界」（與 ruflo / addyosmani / open-agents 三邊比對一致）。

---

## 3. 對 Owner 各 project 的 leverage 矩陣

| Project | leverage | 原因 |
|---|---|---|
| **passive-income** | 高 | 若做 SaaS 化（影片自動化代客生成），cloud agent + sandbox isolation 是必經路徑；可直接 fork 開干 |
| **taichi-framework** | 中-高 | 治理結構同構參考 + LANDSCAPE.md 補一筆「cloud agent template」象限 |
| **aquafeed** | 中 | 70+ 報告查詢 + 給代工廠/客戶 web access 需要外部認證+sandbox，open-agents 的 Better Auth + Neon branch 是現成解 |
| **pid-solver** | 中 | 若做成「貼車輛 spec → 拿 PID」SaaS，open-agents 的 sandbox-per-session 模式直接對應 |
| **scal3r** | 中 | 商品化階段需 cloud upload + isolated processing，sandbox 模型適配 |
| **omniverse** | 低-中 | Isaac Sim 重 GPU，雲端 agent 模式不直接落，但 web 控制台可以借 |
| **soramimi** | 低 | 主要研究階段，web/cloud 不是當前瓶頸 |
| **usv** | 低 | 硬體+本地，與 cloud 路線正交 |
| **aidc-aoi** | 低 | 工廠內網部署，cloud 反而是阻力 |
| **cucumber-detection** | 低 | 已封存負結果為主 |
| **esp-blast** | 低 | 嵌入式硬體 |
| **thesis** | 低 | 學術論文 |
| **local-distillation** | 0 | 本地小模型路線，反向 |

**最高 ROI 配對**：passive-income SaaS 化階段直接 fork open-agents 起家，不重造輪子。其次 aquafeed 客戶/代工廠 portal。

---

## 4. 全面風險評估（5 類）

### 4.1 技術風險：MEDIUM
- **Vercel Workflow SDK 鎖定**：durable workflow 是 Vercel 私有 API（Cloudflare Durable Objects / Inngest / Temporal 是替代但不相容）
- **Sandbox 也是 Vercel 私有**：snapshot-based VM 非 OSS 標準
- **控制措施**：fork 後優先抽象 workflow 層為 interface（Inngest 是最近的 OSS 替代）；或直接接受 vendor-lock 換速度

### 4.2 商業授權風險：LOW
- MIT，可商用、可閉源、可改名
- **但**：依賴的 Vercel 服務有獨立計費（Workflow + Sandbox + KV + Functions），fork 後不上 Vercel 等於砍掉 70% 功能
- **控制措施**：商業化前評估 Vercel pricing；若量大考慮自建 Inngest+Firecracker 替代（成本高但無 lock）

### 4.3 路徑依賴鎖定風險：HIGH（最大風險點）
- Vercel 平台、Neon DB、Better Auth、ElevenLabs 四個 SaaS 一起綁
- 任一斷供 = 全鏈崩
- **控制措施**：(a) 不直接用，只讀架構；(b) 若用 必先做 exit plan（把 workflow 抽象 / DB 改 self-hosted Postgres / Auth 換 Lucia）

### 4.4 政治與供應鏈風險：MEDIUM
- Vercel 是美國公司，對台灣使用者目前無管制
- GitHub App 認證鏈經過 GitHub（同樣美國），passive-income 若做東亞市場無此風險
- **控制措施**：商業客戶在中國大陸時須評估 Vercel/GitHub 不可達；fallback 走自建

### 4.5 社群知識資源風險：LOW
- 5k stars / 636 forks / 活躍 push、官方 vercel-labs 出品
- bus factor 不是 1（lab 團隊維護，不是個人）
- 風險：reference app 性質意味 vercel-labs 可能 freeze 後不更新（template 不是 product）
- **控制措施**：fork 後不期待 upstream 持續維護，視為一次性骨架

---

## 5. 自洽校核（vs Owner memory）

### ✅ 對齊（5 條）
- **infra_sync_model.md**（RPi5 為 git 中央）— open-agents 的「Vercel-as-platform」與共腦「Gitea-as-truth」是同構的「single source 模式」，治理思維一致
- **infra_claude_creative_connectors.md**（passive-income Phase 3 將整合 Adobe/Blender connector）— SaaS 化路徑下 open-agents 是 web 殼候選
- **project_passive_income_allgen.md**（純 AI 生成 + Owner 懶惰=第一性）— SaaS 化能進一步放大「懶惰槓桿」
- **feedback_inventory_owner_tools.md**（Owner 付費工具用到底）— Owner 已有 Claude Max + Adobe，未付 Vercel；採用前要評估 cost
- **feedback_full_autonomy.md**（自動化運維直接做）— open-agents 把 agent 當 background process，與「不問 Owner」哲學一致

### ⚠️ 注意（4 條）
- **feedback_use_scripts.md**（會重複的腳本化）— open-agents 的 skills-lock.json 是 lock 機制，共腦目前 lesson/skill 無 lock，重複會發生
- **project_local_distillation.md**（本地蒸餾備援）— cloud 路線與本地韌性是兩條腿，採 open-agents 不能取代 local
- **project_skill_metabolism.md**（5 substrate + birth filter）— open-agents 13 skill 是 flat 結構，採用其 skill 格式時要保留共腦 lifecycle 機制
- **infra_dataset_cache.md**（中央 dataset 快取）— sandbox VM 模式不適合大 dataset，視覺/影像 pipeline 不能照搬

### ❌ 衝突（2 條）
- **infra_remote_control.md**（RPi5 為 claude.ai 環境，實機執行）— open-agents 是 cloud-only，與 RPi5-first 哲學直接相反；不能整體採用，僅可借結構
- **feedback_session_means_tmux.md**（session = RPi5 tmux）— open-agents 的 session = Vercel workflow run，名同實異，引用時須消歧義

---

## 6. 立即可動手 actions（低承諾、可逆、不付費）

1. **抄 13-skill 命名為共腦 skill candidates 對照表**（30 min）
   產出：`projects/taichi-framework/research/cross-ref-skills-2026-05-08.md`，列共腦目前 0 skill / open-agents 13 skill / addyosmani 20 skill 三邊命名法對照，找命名收斂模式（不建立新 skill，只盤點）

2. **LANDSCAPE.md 補 cloud-agent 象限**（15 min）
   現有 LANDSCAPE 多以 governance/skills 切；補一條「cloud-agent template」象限，列 vercel-labs/open-agents 為標竿，標明與共腦正交（cloud vs local-first）

3. **補 reference memory**（10 min）
   `reference_vercel_open_agents.md`：記錄「durable workflow + ephemeral sandbox + agent host」三分模式，passive-income SaaS 化時的起點 reference

4. **xray-skills.md 加關鍵字**（5 min）
   `durable workflow agent`、`agent sandbox isolation`、`skills-lock`、`vercel sandbox snapshot`、`agent outside vm`

5. **research/REFERENCES.md 加引用**（5 min）
   open-agents URL + MIT + Vercel 官方背書，作為 cloud-agent 章節 anchor

**總計** ≤ 65 分鐘，全部低風險可逆，無新硬體無新付費。

---

## 7. 結論（含 But-For 推導）

**整體判斷：值得低度投入（reference-only）**，不 fork、不採用為基礎設施，但作為「passive-income 若走 SaaS 化的起點骨架」掛一筆。

**But-For 推導**：
- **若沒有 open-agents** → passive-income 走 SaaS 時 Owner 要從零搭 web+auth+workflow+sandbox，至少 2-4 週 → open-agents 把這段壓到「fork + 改 prompt + 部署」≤ 1 天
- **若沒有 cloud 路線** → Owner 永遠 RPi5 + tmux 沒問題，open-agents 完全不需要 → **這是「現在不需要、未來 Phase 3 可能需要」的工具，不是「現在解痛點」的工具**
- **若 Owner 不做 SaaS** → open-agents 的 leverage 降到僅「結構同構參考」，跟 ruflo / addyosmani 同等級

**最終定位**：cloud-agent 對照組標竿、passive-income SaaS 化候選骨架、共腦 skill metabolism 借鑑驗證樣本。**不影響本週焦點（thesis / usv）**。掛 memory + LANDSCAPE 補位 + xray 關鍵字三個低承諾動作即可。

---

## 來源清單

- gh API metadata：https://github.com/vercel-labs/open-agents（2026-05-08 查證）
- README：https://github.com/vercel-labs/open-agents（2026-05-08 WebFetch）
- AGENTS.md：https://github.com/vercel-labs/open-agents/blob/main/AGENTS.md（2026-05-08 WebFetch）
- 目錄結構：https://github.com/vercel-labs/open-agents/tree/main（2026-05-08 WebFetch）
- 13 skill 列表：https://github.com/vercel-labs/open-agents/tree/main/.agents/skills（2026-05-08 WebFetch）
