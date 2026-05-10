# 環境建置

> 最低門檻開始，按需擴展。

## 最低需求

- 支援 system prompt 的 AI 工具（如 Claude Code）
- 持久記憶功能（跨對話保留資訊）
- Git（版本控制）

## 建議配置

- 常駐伺服器（如 Raspberry Pi）：讓 AI 可以在背景執行長任務
- SSH 存取：遠端操作多台裝置
- Bash 自動化：腳本化重複操作

## 不需要

- GPU：框架本身不做訓練或推理
- 雲端服務：所有資料在本地或自有伺服器
- 額外付費 API：只需要你已有的 AI 工具

## 建置步驟

### 1. 取得框架

```bash
git clone <repo-url> wuji-my-axiom
```

### 2. 複製模板到你的專案

```bash
cp wuji-my-axiom/templates/CLAUDE.md your-project/CLAUDE.md
cp wuji-my-axiom/templates/MEMORY.md your-project/MEMORY.md
cp -r wuji-my-axiom/templates/memory your-project/memory
```

`CLAUDE.md` 放在專案根目錄，AI 工具會自動載入作為 system prompt。

### 3. 冷啟動校準

開啟 AI 對話，讓它讀取校準指南：

```
請讀取 calibration/GUIDE.md 並開始校準流程。
```

校準流程會問你約 10 個問題，分三層遞進：
1. 找出你的核心驅動與限制（兩儀）
2. 展開你的行為模式（四象）
3. 從行為壓縮出你的公理（太極）

詳細流程見 `calibration/GUIDE.md`。

### 4. 填入「我的卦」

校準完成後，AI 會產出你的兩儀、四象與應用層太極。把結果填入 `CLAUDE.md` 的「我的卦」區塊：

```markdown
## 我的卦

兩儀：{你的核心驅動} / {你的核心限制}
四象：{四種行為模式}
應用層太極：{你的個人公理}
```

### 5. 開始使用

正常使用 AI 工具。框架會透過變通機制自動校準：
- 你糾正 AI 時，它記住錯誤
- 你確認 AI 時，它強化做法
- 累積的模式逐漸升級為爻則

## 支援的 AI 工具

**主要支援**：Claude Code
- 原生支援 CLAUDE.md 作為 system prompt
- 內建持久記憶（memory）
- 完整 bash 環境

**可適配**：任何支援以下功能的 AI 工具
- 自訂 system prompt（放入 CLAUDE.md 內容）
- 跨對話持久記憶（或手動管理記憶檔案）
- 檔案讀寫能力

適配時需手動處理：記憶檔案的讀寫、啟動流程的觸發、變通機制的執行。核心邏輯不變，只是介面不同。

## 驗證安裝

安裝完成後，開啟新對話，AI 應該：
1. 自動載入 CLAUDE.md
2. 讀取 MEMORY.md
3. 報到格式如下：

```
上線。
道：正常
德：0 條待處理
器：正常
```

看到這個輸出，表示框架已正確運作。
