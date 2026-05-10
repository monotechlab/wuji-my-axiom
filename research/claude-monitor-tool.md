# Claude Code Monitor Tool 研究報告

> 調研日期：2026-04-12
> 目的：評估 Monitor tool 對共腦架構的影響，特別是能否取代現有的 polling 機制

---

## 1. Monitor Tool 是什麼

Claude Code v2.1.98（2026-04-09）新增的內建工具。核心概念：**背景腳本的 stdout 變成事件流，每一行輸出都是一個通知，觸發 Claude 反應。**

與現有工具的區別：

| 工具 | 行為 | 適用場景 |
|------|------|---------|
| `Bash run_in_background` | 跑完通知一次 | 一次性任務（build 完了告訴我） |
| `Monitor` | 每行輸出都通知 | 持續監控（出錯的瞬間告訴我） |
| `CronCreate` | 定時觸發 prompt | 週期性任務（每小時跑一次） |
| `Hooks` | 工具調用前後執行 | 護欄/驗證 |

**關鍵特性：靜默零成本。** 沒事件 = 零 token 消耗。有事件 = 只處理那一行。

## 2. 技術規格

### 參數

| 參數 | 用途 | 預設 |
|------|------|------|
| `description` | 通知標籤 | 必填 |
| `command` | 產生事件的 shell 腳本 | 必填，stdout 每行 = 一個事件 |
| `timeout_ms` | 自動終止時間 | 300,000ms（5 分鐘），最大 3,600,000ms（1 小時） |
| `persistent` | 是否跟隨 session | `true` = 整個 session 存活，用 `TaskStop` 停 |

### 事件批次

200ms 內到達的多行輸出會批次成單一通知。

### 兩種監控模式

**Stream Filter（流式過濾）：**
```bash
# 監控應用日誌
tail -f /var/log/app.log | grep --line-buffered "ERROR"

# 監控檔案變更
inotifywait -m --format '%e %f' /watched/dir

# WebSocket 事件
node watch-for-events.js
```

**Poll-and-if Filter（輪詢過濾）：**
```bash
# 每 30 秒檢查 PR 狀態
last=$(date -u +%Y-%m-%dT%H:%M:%SZ)
while true; do
  now=$(date -u +%Y-%m-%dT%H:%M:%SZ)
  gh api "repos/owner/repo/issues/123/comments?since=$last" \
    --jq '.[] | "\(.user.login): \(.body)"'
  last=$now; sleep 30
done
```

### 關鍵規則

1. **必須用 `grep --line-buffered`** — 否則 pipe buffering 延遲事件數分鐘
2. **處理暫時性失敗** — API 呼叫後加 `|| true`，單次逾時不能殺掉整個 monitor
3. **stdout 要精簡** — 每行都變對話訊息，過多事件會自動停止

### 限制

- 需要 Claude Code v2.1.98+
- 不支援 Amazon Bedrock、Google Vertex AI、Microsoft Foundry
- 權限規則同 Bash tool
- 最長 1 小時（persistent 模式除外）

## 3. 對共腦架構的影響

### 3.1 現有架構（polling-based）

```
system-heartbeat.sh (cron */5)
    ├→ TODO hash 比對 → tmux send-keys brain "[EVENT] TODO_CHANGED"
    ├→ 殭屍 worker 檢查 → tmux send-keys brain "[EVENT] WORKER_ZOMBIE"
    ├→ periodic nudge (每 6 小時) → tmux send-keys brain "[EVENT] PERIODIC_NUDGE"
    └→ memory health check

brain-event.sh (inotifywait)
    └→ TODO.md 修改 → tmux send-keys brain "[EVENT] TODO_CHANGED"

brain-resurrect.sh (cron */30)
    └→ brain 死了 → 重啟
```

**問題：**
- heartbeat 每 5 分鐘跑一次，大部分時候什麼都沒發生 = 浪費
- brain-event.sh 是獨立 process，與 brain session 無直接通訊管道
- tmux send-keys 是脆弱的 IPC（送字串到終端，可能干擾正在進行的對話）

### 3.2 Monitor 能改什麼

| 現有機制 | Monitor 替代方案 | 效益 |
|---------|-----------------|------|
| brain-event.sh (inotifywait) | brain 自己跑 `Monitor(inotifywait -m projects/*/TODO.md)` | 省一個獨立 process，事件直接進 brain context |
| heartbeat TODO hash 比對 | 同上，inotifywait 即時性更好 | 從 5 分鐘延遲 → 即時 |
| worker-done 偵測 | `Monitor(inotifywait -m projects/*/.worker-result.json)` | Worker 完成瞬間 brain 就知道，不用等 heartbeat |
| 殭屍 worker 檢查 | `Monitor(while true; do tmux ls ...; sleep 300; done)` | 可行但不比 heartbeat 好太多 |
| periodic nudge | `CronCreate` 更適合（定時觸發） | Monitor 不適合純定時任務 |

### 3.3 具體改造方案

**Phase 1：brain 啟動時自動掛 Monitor（低風險，高效益）**

在 `start-brain.sh` 的 prompt 中加入指令：

```
## 啟動後立即執行
啟動後用 Monitor tool 掛兩個監控：

1. TODO 變更監控：
   Monitor({
     description: "TODO file changes",
     command: "inotifywait -m -e modify -e close_write --format '%f' projects/*/TODO.md 2>/dev/null || true",
     persistent: true
   })

2. Worker 完成監控：
   Monitor({
     description: "Worker result files",
     command: "inotifywait -m -e modify -e close_write --format '%w%f' projects/*/.worker-result.json 2>/dev/null || true",
     persistent: true
   })

收到 TODO 事件 → 讀該專案 TODO.md，判斷是否派手
收到 worker-result 事件 → 讀結果，決定下一步
```

**Phase 2：簡化 heartbeat（中風險，中效益）**

brain 自己監控了 TODO 和 worker-result 後，heartbeat 可以移除：
- ~~TODO hash 比對~~（Monitor 取代）
- ~~brain-event.sh~~（Monitor 取代）
- 保留：殭屍清理、health check、memory check、system.json 生成

**Phase 3：全面事件驅動（高風險，需驗證）**

```
brain session
├→ Monitor: TODO 變更 (inotifywait)
├→ Monitor: worker 完成 (inotifywait)
├→ Monitor: 日誌錯誤 (tail -f | grep ERROR)
├→ CronCreate: periodic nudge (每 6 小時)
└→ CronCreate: health check (每小時)

heartbeat 退化為：
├→ brain 存活檢查 + 重啟
├→ system.json 生成
└→ 殭屍清理
```

### 3.4 風險

1. **Monitor 穩定性未知** — 2026-04-09 剛發布，生產環境經驗不足
2. **brain session 重啟 = Monitor 全失** — persistent Monitor 綁定 session 生命週期。brain 重啟後需要重新掛載所有 Monitor
3. **事件過多可能自動停止** — 文件說「excessive events auto-stop」，高活躍專案可能觸發
4. **inotifywait 在 RPi5 上的穩定性** — 需要驗證 ARM64 + 多目錄監控的表現

### 3.5 與 Hermes 調研的關聯

剛完成的三項 Hermes 啟發機制：

| 機制 | 現有實作 | Monitor 可改進 |
|------|---------|---------------|
| Skill Extraction | worker cleanup 呼叫 skill-extract.sh | Monitor worker-result → 自動觸發，不需在 cleanup 腳本中 |
| Memory Check | heartbeat 每 5 分鐘跑 | 不需要改，頻率夠低 |
| Periodic Nudge | heartbeat 每 6 小時送事件 | 改用 CronCreate 更乾淨（brain 內建排程，不依賴外部 heartbeat） |

## 4. 建議

### 立即可做（本週驗證週）

1. **升級 Claude Code** 到 v2.1.98+（確認 Monitor 可用）
2. **在 brain prompt 中加 Monitor 指令**：TODO 變更 + worker-result 監控
3. **保留 heartbeat 作為 fallback** — Monitor 失敗時 heartbeat 還能兜底

### 後續（驗證通過後）

4. 移除 brain-event.sh（Monitor 取代）
5. 簡化 heartbeat（移除 TODO hash 比對）
6. periodic nudge 改用 CronCreate

### 不建議

- 完全移除 heartbeat — 它還負責 system.json 生成、殭屍清理等 Monitor 不適合的工作
- 用 Monitor 監控所有東西 — 保持「大部分極保守，小部分極激進」

## 5. 結論

**Monitor tool 是共腦架構從 polling 到 event-driven 的關鍵拼圖。** 它解決了 tmux send-keys 的脆弱性問題，讓 brain 自己持有監控能力而不依賴外部腳本注入事件。

與 Hermes 研究的結合：Hermes 的 Periodic Nudge 是定時自省，Monitor 是事件驅動反應。兩者互補——定時自省用 CronCreate，即時反應用 Monitor。

**優先級：先掛 Monitor（TODO + worker-result），驗證穩定後再簡化 heartbeat。**

---

## Sources

- [Claude Code Tools Reference](https://code.claude.com/docs/en/tools-reference)
- [Monitor Tool Guide (claudefast)](https://claudefa.st/blog/guide/mechanics/monitor)
- [Claude Code Changelog](https://code.claude.com/docs/en/changelog)
- [Blocktempo: Monitor Tool 報導](https://www.blocktempo.com/claude-code-monitor-tool-background-streaming-replaces-polling-saves-tokens/)
- [Aiia: Claude Code Background Scripts](https://aiia.ro/blog/claude-code-monitor-tool-background-scripts/)
