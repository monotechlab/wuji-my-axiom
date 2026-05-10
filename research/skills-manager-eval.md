# xingkongliang/skills-manager — 深度評估

> 2026-05-08 · clone 自 https://github.com/xingkongliang/skills-manager (commit 由 `git pull` 至當天 head 為止 281 commits)

## TL;DR

skills-manager 是 **skill artifact 的「分發 + 版本化」基礎設施**，不是 lifecycle 系統。它解決「同一份 SKILL.md 怎麼同時出現在 Cursor / Claude Code / Codex / 14+ 工具」的工程瑣事，用 symlink/copy + 中央 git repo + SQLite 中繼資料完成。

對 Owner 的 skill metabolism 機制是**正交互補**而非衝突：
- skills-manager 管 **where**（distribution、version、cross-tool path mapping）
- 共腦 metabolism 管 **whether / how long**（birth filter、usage tracking、demote）

但**不建議現階段整合**，因為 `~/.claude/skills/` 目前是空的（0 個 SKILL.md），分發層解的是還不存在的問題。先讓 birth filter 產出 ≥3 個 skill 後再評估整合。

---

## 1. 基本資料

| 項目 | 值 | 出處 |
|---|---|---|
| Stars | 1196 | GitHub API 2026-05-08 |
| Forks | 110 | GitHub API |
| Open issues | 28 | GitHub API |
| 建立日期 | 2026-03-02 | GitHub API `created_at` |
| 最後 push | 2026-05-08 (今天) | GitHub API `pushed_at` |
| Commit 總數 | 281 | `git rev-list --count HEAD` |
| 月活躍度 | 2026-03: 175 / 2026-04: 98 / 2026-05: 8 | `git log --format=%ai` |
| License | MIT | `LICENSE` |
| 主要作者 | Tianliang Zhang (xingkongliang / Jay.TL) — 254 / 281 commits ≈ 90% | `git shortlog -sne` |
| 第二作者 | enshulv (9), whereissam (7), samshine (3) | 同上 |
| 對外身份 | @JayTL00 on X, buymeacoffee.com/jaytl | `README.md:14-17` |

**社群活躍度判讀**：3-month-old repo，1196 ★ 表示熱度極高（XRAY 抓到的 1049 是 4 天前，48hr 還在漲）。但作者分佈高度集中（~90% 主作者），bus factor = 1。今天還在 push，活著。

XRAY 的「Rust」標籤正確（`src-tauri/Cargo.toml:6` `edition = "2021"`），但實際是 **Tauri 2 desktop app + Rust CLI 共享 core**，不是純 CLI 工具。

## 2. 核心功能

### 2.1 架構分層

```
┌─────────────────────────────────────────────────────┐
│  Frontend: React 19 + TS + Vite + Tailwind          │  src/
│  ├─ Tauri desktop (npm run tauri:dev)               │
│  └─ —————— invoke ——————————————                     │
├─────────────────────────────────────────────────────┤
│  Tauri commands layer (Rust)                         │  src-tauri/src/commands/
│  └─ skills.rs (2006 lines), scenarios.rs, sync.rs   │
├─────────────────────────────────────────────────────┤
│  Core lib (Rust, headless reusable)                  │  src-tauri/src/core/
│  ├─ tool_adapters.rs   ── 49 個工具的 path 對照表     │
│  ├─ sync_engine.rs     ── symlink / copy 同步        │
│  ├─ scenario_service.rs── scenario → tool target 解析│
│  ├─ scanner.rs / installer.rs / skill_metadata.rs    │
│  ├─ git_backup.rs / git_fetcher.rs                   │
│  ├─ skill_store.rs     ── SQLite (rusqlite, 1179 ln) │
│  └─ skillsmp_api.rs    ── skills.sh marketplace      │
├─────────────────────────────────────────────────────┤
│  CLI binary: skills-manager-cli                     │  src-tauri/src/bin/skills-manager-cli.rs
│  └─ 共用同一份 core，--json 輸出 agent-friendly      │
└─────────────────────────────────────────────────────┘
   ↓ 寫入磁碟
   ~/.skills-manager/         ── 中央 repo (預設)
     ├─ skills/               ── 真正的 SKILL.md 內容（git versioned）
     └─ skills-manager.db     ── SQLite metadata（不入 git）
```

CLI 路徑（`README.md:118-150`）：
- `repo status / set-path` — 切換中央 repo
- `tools list` — 列已安裝工具
- `skills list / show / export` — skill CRUD
- `scenarios list / preview / apply <name>` — **核心同步動作**
- `git status / pull / push / commit` — 中央 repo 版本控制

### 2.2 跨工具同步邏輯（重點）

**ToolAdapter 結構**（`src-tauri/src/core/tool_adapters.rs:5-26`）：
```rust
pub struct ToolAdapter {
    pub key: String,                       // "claude_code"
    pub display_name: String,              // "Claude Code"
    pub relative_skills_dir: String,       // ".claude/skills"
    pub relative_detect_dir: String,       // ".claude"  (用來判斷工具有沒有裝)
    pub additional_scan_dirs: Vec<String>,
    pub override_skills_dir: Option<String>,
    pub is_custom: bool,
    pub recursive_scan: bool,
}
```

**寫死的工具表**（`tool_adapters.rs:117-…`，數出 49 個 `key:`）：
cursor, claude_code, codex, opencode, antigravity, amp, kilo_code, roo_code, goose, gemini_cli, github_copilot, openclaw, droid, windsurf, trae, cline, deepagents, firebender, kimi, replit, warp, augment, bob, codebuddy, command_code, continue, cortex, crush, iflow, junie, kiro, kode, mcpjam, mistral_vibe, mux, neovate, openhands, pi, pochi, qoder, qwen_code, trae_cn, zencoder, adal, hermes（+ 4 個 plus 1 個 helper struct）。
README 寫「15+」，實際 ≈49。社群正在追新工具。

**同步動作**（`src-tauri/src/core/sync_engine.rs:60-110`）：
1. 為每個 enabled skill 算出 `(source: ~/.skills-manager/skills/<skill>, target: ~/<tool_dir>/skills/<skill>)`
2. 預設 symlink，Windows 失敗 fallback copy（`sync_engine.rs:79-95`）
3. `is_target_current()` 走 `read_link` 確認 symlink 指向正確才跳過，否則 remove + redo
4. `ensure_dst_not_inside_src()` 防止 dst 在 src 子樹內導致無限遞迴 copy（issue #61 的補丁）

**Skill 定義**：directory 內有 `SKILL.md` 或 `skill.md`，YAML frontmatter 有 `name` / `description`（`skill_metadata.rs:32-78`）。**不解析 9 段 anatomy**（addyosmani 那套），只認 frontmatter。

**Scenario** = "一組 enabled skill + per-skill 開放給哪些 tool" 的命名集合。`scenarios apply Default` = 把 Default 集合解析成 sync targets 後跑 `sync_engine`。

### 2.3 安裝來源（4 種）

`installer.rs:20-60` 看到：
- 本地目錄
- `.zip` / `.skill` 壓縮檔
- Git URL（透過 `git2` crate）
- skills.sh (skillsmp.com) marketplace（`skillsmp_api.rs`，要 API key）

中央 repo 自身用 git 版控（`core/git_backup.rs`），可設 remote 做多機同步，每次 `Sync to Git` 自動建 `sm-v-<timestamp>-<sha>` 形式的 snapshot tag（`git_backup.rs:50`），可 restore。

## 3. 與共腦 skill metabolism 對照

> 引共腦 memory `project_skill_metabolism.md`：5 層 substrate（inline / memory rule / script / skill / agent）+ birth filter + demotion + weekly audit。

| 維度 | skills-manager | 共腦 metabolism |
|---|---|---|
| 處理層 | 只處理「skill」一層 | 5 層 substrate（含 inline/memory/script/agent）|
| 主要動作 | **distribute** — 把 SKILL.md 部署到 N 個工具 | **curate** — 決定該不該存在、何時消滅 |
| 建立規則 | 全憑 user 手動「import + enable」 | birth filter 4 條件（C1 高頻 / C2 編排 / C3 推導錯率 / C4 身份隔離）|
| 使用追蹤 | 無 | `.claude/usage/<type>/<name>.jsonl` |
| 降格邏輯 | 無 | agent→skill→memory→archive，30/90/180d 階梯 |
| 週審 | 無 | rpi5 cron `skill-audit-weekly.sh` |
| 跨工具部署 | **49 工具 path 對照 + symlink 引擎** | 無（共腦目前綁 Claude Code 一條路）|
| 跨機器同步 | 中央 git repo + remote push/pull | 走 Gitea/git pull-rebase（同邏輯，但無 skill-aware）|
| 版本快照 | 自動建 sm-v-* tag + restore UI | 無 skill-level 版本，靠整 repo git history |

**一句話結論**：兩者解的是不同維度的問題。skills-manager 是**橫向擴散**（一份 → 多工具/多機器），metabolism 是**縱向篩選**（多想法 → 該存活的少數）。整合 = 在篩選通過後接上分發。

**addyosmani 調研時下的結論依舊成立**：skills-manager 跟 addyosmani 一樣，**沒有 lifecycle**。共腦在 lifecycle 維度仍領先。

## 4. 直接整合可行性（rabot RPi5 + Tailscale）

### 可行性 ✓
- **Rust toolchain on rabot**：可裝（aarch64 有 prebuilt toolchain）
- **CLI 安裝命令**：`cargo install --path src-tauri --bin skills-manager-cli --locked --force`（`README.md:175`）→ 落地 `~/.cargo/bin/skills-manager-cli`
- **無頭可用**：CLI 不需要 GUI，`--json` 旗標方便 worker 解析
- **中央 repo 與 Gitea 共存**：`~/.skills-manager/skills/` 設 remote 為 `http://192.168.0.142:3000/ra/skills.git`，與 ai-group 平行管理，不污染本 repo

### 摩擦點 ✗
1. **Owner 目前 0 個 skill**（`find ~/.claude/skills/ -name SKILL.md` = 0）。先解分發是過早最佳化。
2. **Tauri 桌機 app 無用武之地**：rabot 無 GUI，desktop app 不能跑；只能用 CLI。少了 marketplace browse / diff viewer 的 UX 價值。
3. **Tailscale 不必要**：rabot/laptop/desktop 已用 Gitea LAN（192.168.0.142）+ Tailscale fallback，skills-manager 走 git remote 即可，不需要新通道。
4. **重定義「scenario」**：skills-manager 的 scenario = 一組 enabled skill。共腦現有概念是「焦點專案」+ CLAUDE.md，再疊一層 scenario 增加心智負擔。**疊則必須廢一個**。
5. **bus factor = 1**：作者一人推 90% commits，repo 才 3 個月。鎖定它前要評估永續性。
6. **49 工具表寫死**：新工具要等作者更新或自己 fork。Owner 用的 Claude Code 已內建（`tool_adapters.rs:128-135`）。

### 建議用法
- **不 fork**，借鑑 ToolAdapter 結構即可（path mapping table 是公知資訊）
- **CLI 模式試用**，桌機 app 略過
- **延後決策**到共腦累積 ≥3 個 skill 之後

## 5. 立即可動手的 3 個 actions

### Action 1（30 min）— Headless CLI smoke test
**目的**：驗證 rabot 上能不能跑、有沒有版本相依坑。

```bash
# 在 rabot
cd /tmp
git clone https://github.com/xingkongliang/skills-manager.git
cd skills-manager
cargo build --release --manifest-path src-tauri/Cargo.toml --bin skills-manager-cli
./src-tauri/target/release/skills-manager-cli --json repo status
./src-tauri/target/release/skills-manager-cli --json tools list
```

驗收：`tools list` 應列出 claude_code 為 detected（`~/.claude/` 存在）。
失敗預期：rust-version = 1.77.2，rabot 上 rustc 若 < 1.77 要先升。

### Action 2（45 min）— 本機單向 sync demo（**這是 sync demo，可跑**）
**目的**：實際看到「中央 SKILL.md → ~/.claude/skills/」的 symlink。

```bash
# 1. 在 rabot 建一份 demo skill（不是真正共腦 skill，純驗證）
mkdir -p ~/.skills-manager/skills/demo-hello
cat > ~/.skills-manager/skills/demo-hello/SKILL.md <<'EOF'
---
name: demo-hello
description: smoke-test skill, delete after verification
---
# Hello
This is a sync demo.
EOF

# 2. 用 CLI 把 demo-hello 同步到 claude_code
~/.cargo/bin/skills-manager-cli skills list --json
~/.cargo/bin/skills-manager-cli scenarios list
# 若 Default scenario 不存在，先用桌機 app 建（或寫 SQLite seed），這是個摩擦點

# 3. 驗收
ls -la ~/.claude/skills/demo-hello/
readlink ~/.claude/skills/demo-hello   # 應指向 ~/.skills-manager/skills/demo-hello
```

**注意**：`scenarios apply` 需要先有 scenario 記錄在 SQLite。若 CLI 沒提供 `scenario create`，必須先跑桌機 app 建 → 這是 headless 路線的最大坑，**Action 2 部分需要回頭看 commands/scenarios.rs 確認**。如果 CLI 真的不能憑空建 scenario，demo 改寫直接呼叫 `skill-export` 命令繞過 scenario：
```bash
skills-manager-cli skills export demo-hello --dest ~/.claude/skills/demo-hello
```
（`README.md:138` 確認此命令存在）

### Action 3（1 hr，**先不做，等條件成立**）— Multi-machine sync via Gitea
**前提**：Action 1 + 2 通過 + 共腦累積 ≥3 個真正用得上的 skill。

```bash
# rabot
cd ~/.skills-manager/skills
git init && git add . && git commit -m "init"
git remote add origin http://192.168.0.142:3000/ra/skills.git
git push -u origin main

# laptop（透過 ssh laptop）
git clone http://192.168.0.142:3000/ra/skills.git ~/.skills-manager/skills
cargo install --git https://github.com/xingkongliang/skills-manager skills-manager-cli  # 先在 laptop 裝
skills-manager-cli scenarios apply Default
```

**為何延後**：分發 0 個 skill = 沒有意義的同步。先過 birth filter。

---

## 不做的事（負面決策）

- 不 fork。MIT 授權允許，但 90% bus factor + 跟 Owner 5-substrate 模型相容性低，fork 維護成本 > 借鑑 path table 成本。
- 不裝桌機 app。rabot 無 GUI，laptop/desktop 雖有 GUI 但 Owner 流程是 CLI/SSH，桌機 app 是負加值。
- 不採用 scenario 概念進入共腦。心智模型衝突（共腦已有「焦點專案」+ memory rule），疊加會碎片化。
- 不在共腦 skill metabolism 設計中讓步給 skills-manager。lifecycle 機制是共腦差異化點，不放棄。

## 引用

- 倉庫：https://github.com/xingkongliang/skills-manager  · clone 路徑 `/tmp/skills-manager-full/` (commit head 至 2026-05-08)
- ToolAdapter 結構：`src-tauri/src/core/tool_adapters.rs:5-26`
- 49 工具枚舉：`tool_adapters.rs:117` 起 `default_tool_adapters()`
- Sync 引擎：`src-tauri/src/core/sync_engine.rs:60-180`
- Skill metadata 解析：`src-tauri/src/core/skill_metadata.rs:32-78`（只認 YAML frontmatter，無 anatomy）
- CLI 入口：`src-tauri/src/bin/skills-manager-cli.rs:1-80`
- README CLI 章節：`README.md:118-180`
- 共腦對照來源：memory `project_skill_metabolism.md`
- 上一輪相關調研：`research/addyosmani-agent-skills.md`（2026-05-07）

## 不確定 / 未驗證項

- [未驗證] CLI `scenarios apply` 在 SQLite 無 scenario 紀錄時的行為；若需先用桌機 app 種 seed → headless 路線會卡。Action 2 跑了才會知道。
- [未驗證] rabot rustc 版本 ≥ 1.77.2。
- [未驗證] aarch64 上 `git2` 帶 `vendored-openssl` feature 的編譯時間（笨重，可能 5-10 min）。
- [未驗證] skills.sh marketplace 實際品質——僅程式碼有引用 https://skillsmp.com 端點，未瀏覽內容。
