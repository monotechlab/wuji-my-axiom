# khazix-skills 調研報告

> 來源：https://github.com/KKKKhazix/khazix-skills  
> 調研日期：2026-04-22  
> 調研者：Mono（共腦）

---

## 一、Repo 基本資訊

| 項目 | 值 |
|---|---|
| Owner | KKKKhazix（數字生命卡兹克）|
| 語言 | Python 100% |
| Stars / Forks | 5,807 / 978 |
| License | MIT（Copyright 2026）|
| 建立 | 2026-04-06 |
| 最後更新 | 2026-04-18 |
| 狀態 | 活躍，非歸檔 |
| Repo 大小 | 39 KB |

**活躍度評估：** 高。建立至今僅 12 天，已達 5k+ stars，說明作者有影響力且內容有市場。提交紀錄顯示 Claude Opus 共著，工作流與我們的共腦模式相近。

---

## 二、Skill 清單與用途

### 2.1 hv-analysis（橫縱分析法）

**用途：** 對產品、公司、技術、人物進行自動化深度研究，產出 10,000–30,000 字的 PDF 報告。

**結構：**
- 縱軸（歷史發展）：6,000–15,000 字
- 橫軸（競品比較）：3,000–10,000 字
- 交叉分析（合成洞察）：1,500–3,000 字

**技術實作：**
- `SKILL.md`（19.7 KB）—— skill 定義本體
- `scripts/md_to_pdf.py` —— WeasyPrint 轉 PDF，支援中文、A4、頁首頁尾、封面
- `references/schema.json` —— 資料結構 schema

**觸發語：** "幫我研究"、"deep dive"、"競品分析"、"這個怎麼回事"

---

### 2.2 khazix-writer（長文寫作 Skill）

**用途：** 以卡兹克個人風格產出微信公眾號長文，強調「知識分子普通人認真討論打動自己的事」的文風。

**五種文章類型：**
1. 調查實驗（第一手體驗）
2. 產品體驗（親測）
3. 現象分析（觀察→洞察）
4. 工具分享（敘事式）
5. 方法論（經驗型可操作框架）

**關鍵機制：**
- 四層質量保證（L1 硬規則 → L2 風格一致性 → L3 內容深度 → L4 人味主觀審查）
- 禁用語清單（「說白了」「意味著什麼」冒號破折號等 AI 痕跡）
- 節奏控制（短句長句交替 + 定期 keystone sentence）

---

### 2.3 prompts/橫縱分析法.md（輕量提示詞版）

**用途：** hv-analysis 的簡化版本——不含 Python 腳本，只是可貼上的研究框架提示詞。

---

## 三、與我們現有 Skills/ 的比對

### 我們目前的狀態

| 層面 | 現況 |
|---|---|
| `~/.claude/skills/` | 空目錄，無自訂 skill |
| `ai-group/skills/` | 不存在 |
| Claude Code 內建 skills | update-config / simplify / loop / schedule / claude-api / init / review / security-review 等 |
| TODO 計畫 | `addyosmani/agent-skills` 移植（2026-04-19 標記）|

### 重複分析

| khazix skill | 我們有對應的嗎？ | 判斷 |
|---|---|---|
| hv-analysis（深度研究） | 無。research/*.md 是人工撰寫，無自動化 skill | **互補** |
| khazix-writer（微信長文） | 無。passive-income 需要 AI 生成腳本，方向接近 | **部分互補** |
| 橫縱分析法（提示詞） | 無。LANDSCAPE.md 是靜態文件不是 skill | **互補** |

### 架構模式觀察

khazix-skills 採用 `SKILL.md` 格式，符合 Agent Skills 開放標準（同 `addyosmani/agent-skills`），這與我們 TODO 計畫移植的目標格式一致。這意味著：

- khazix-skills 是 addyosmani/agent-skills 標準的實作範例
- 可作為我們建立 `ai-group/skills/` 時的格式參考
- CLAUDE.md 描述的「skill-build 私域 → 公版鏡像」工作流可直接借鑑

---

## 四、建議

### 4.1 hv-analysis — 建議**借鑑，不 fork**

**理由：**
1. MIT 授權，可自由取用
2. 橫縱分析法是方法論，非卡兹克私有，可通用化
3. 我們的調研任務（LANDSCAPE.md、research/*.md）目前完全靠 Mono 手動執行，一個自動化的深度研究 skill 可大幅降低每次調研的 token 成本
4. PDF 輸出（`md_to_pdf.py`）對太極框架論文初稿有直接用途

**行動：** 將 `SKILL.md` 和 `prompts/橫縱分析法.md` 作為範本，建立 `ai-group/skills/hv-research/SKILL.md`，客製化為太極框架的研究風格（更強調「貞必接地」驗證要求）。

### 4.2 khazix-writer — 建議**有條件借鑑**

**理由：**
1. passive-income 專案需要 AI 生成影片腳本（中文），風格反棒讀要求與 khazix-writer 的四層 QA 高度重疊
2. 但 khazix-writer 高度針對微信公眾號長文風格，與我們的短影音腳本場景有格式差異

**行動：** 不直接移植，借鑑其「禁用語清單 + 四層 QA」機制，建立 `ai-group/skills/tw-script/SKILL.md`（台灣中文短影音腳本生成，含反棒讀規則）。需要時再執行，現在不列入優先。

### 4.3 架構 — 建議**立即採用 SKILL.md 格式**

**理由：**
1. khazix-skills 驗證了 SKILL.md 格式可行且有市場
2. 與 TODO 計畫中 `addyosmani/agent-skills` 移植目標格式一致
3. 建立格式標準後，未來每個 skill 都可複用結構

**行動：** 在 `addyosmani/agent-skills` 移植任務完成前，用 khazix-skills 的 SKILL.md 結構作為臨時標準。

### 4.4 fork — 不建議

**理由：** 內容高度個人化（卡兹克聲音、微信文章），fork 意義不大。MIT 授權允許直接借用片段，不需要 fork 整個 repo。

---

## 五、摘要判斷

| 維度 | 評分（1–5）| 說明 |
|---|---|---|
| 技術品質 | 4 | SKILL.md 結構清晰，Python 腳本實用 |
| 活躍度 | 5 | 12天 5k stars，持續更新 |
| 授權友好性 | 5 | MIT，無限制 |
| 與我們的相關性 | 3 | hv-analysis 高度相關，khazix-writer 有條件相關 |
| 建議行動 | **借鑑 hv-analysis + 格式標準** | fork 無必要 |

---

*來源已驗證：GitHub API + raw file fetch，2026-04-22 存取。*
