# Claude Design（Anthropic 產品）研究報告

> 調研日期：2026-04-21
> 目的：評估 Anthropic Labs 新產品 Claude Design 對 Owner 工作流的應用價值
> 資料來源：Anthropic 官方公告（2026-04-17）、TechCrunch、Inc、iThome 報導（175160）、Claude Help Center

---

## 1. Claude Design 是什麼

**發布日**：2026-04-17，Anthropic Labs（Anthropic 的實驗產品部門）發布。

**定位**：把 Anthropic 從「模型／開發工具」供應商，切進**視覺設計工作流**。直接對標 Figma / Adobe / Canva。

**引擎**：Claude Opus 4.7（與 agentic coding 共用模型）。

**入口**：claude.ai/design

**訂閱綁定**：僅 Claude Pro / Max / Team / Enterprise 可用，Free 不行。使用量計入原訂閱 limits。

**市場反應**：發布當日 Figma 股價大跌。

---

## 2. 核心功能

### 2.1 輸入

- 純文字 prompt
- 上傳圖片
- 文件：**DOCX / PPTX / XLSX**
- 指向 codebase（讀原始碼與設計檔案建立公司 design system）

### 2.2 產出類型

| 類別 | 用途舉例 |
|---|---|
| 簡報 | Pitch deck, 公司介紹 |
| 網站 / Landing page | MVP prototype |
| 產品 wireframe / mockup | Product Manager → 交 Claude Code 實作 |
| 行銷素材 | Landing page、社群貼文、campaign visuals |
| One-pagers | 產品單頁 |

### 2.3 輸出格式

- **PPTX**（可丟進 PowerPoint 繼續改）
- **Canva**（直接送入 Canva 編輯）
- PDF
- Standalone HTML
- 內部 URL（組織內分享）

### 2.4 協作

多人可同時在同個 design 上與 Claude 群組對話、修改。line-level editing 支援。

### 2.5 Design System Onboarding

第一次使用時，Claude 讀 codebase 與既有設計檔案，提取公司專屬色彩、字型、元件，之後產出的設計自動遵守 design system。可手動修正。

---

## 3. 與 Claude 生態的位置

```
┌─────────────────────────────────────────────┐
│ Claude 生態垂直整合（截至 2026-04）          │
├─────────────────────────────────────────────┤
│ 模型：Opus 4.7 / Sonnet 4.6 / Haiku 4.5     │
│ 開發：Claude Code + Agent SDK + API          │
│ 設計：Claude Design（新） ← 這是本報告主題   │
│ 聊天：Claude.ai                              │
│ 記憶：CLAUDE.md + Projects                   │
└─────────────────────────────────────────────┘
```

Anthropic 正從「AI 模型公司」轉向「AI 應用平台公司」。Claude Design 是此轉型的重要一步。

---

## 4. 對 Owner「自身」的改進機會

按時效與價值排序：

### 4.1 碩論／博班面試簡報（4/30 博班面試前急件）

**現狀**：thesis v1.0 送審中 + 4/30 博班面試迫近，簡報製作是下階段主要工作瓶頸。

**Claude Design 路徑**：
- 上傳論文 DOCX（v1.0 主幹）→ Claude Design 直接產出 PPTX
- 對 design system 指定學術風格（低彩度、襯線字、圖表清楚）
- export PPTX 後在 PowerPoint 微調細節（圖表公式）
- 預計節省 4–8 小時

**對照既有工具**：筆電 Adobe 全家桶 + PowerPoint 手做約 6–10 小時。Claude Design + PowerPoint 後期約 1–3 小時。

**風險**：自動產簡報可能不符合 NCHU 論文簡報慣例（需人工把關第一次產出）。

### 4.2 Passive-income 視覺生產（長期）

**現狀**：passive-income 全生成路線，純 AI 生成，不實拍。科普 9:16 + 科技時事 16:9 雙軌格式。封面、縮圖、社群貼文目前仍是手做痛點。

**Claude Design 路徑**：
- 每期影片前，用 Claude Design 批量生 10–20 張封面候選
- 建立 channel 專屬 design system（色彩、字型、風格關鍵字）
- 輸出 PNG 或直接送 Canva 做 A/B test 縮圖

**與 memory 中「Adobe 自動化是業界標準」對話**：
- Adobe 自動化（ExtendScript / aerender / pymiere）是**影片量產**的解（Phase 3）
- Claude Design 是**靜態視覺量產**的新解（Phase 1–2）
- 兩者不衝突，Claude Design 補 ffmpeg 不擅長的設計層

### 4.3 Fay Liu Studio 接案（收入放大）

**現狀**：接案信箱已建，但無快速出 proposal 的流程。

**Claude Design 路徑**：
- 客戶需求 brief → Claude Design 產出 3 版 landing page / logo / 配色方案
- 節省客戶決策週期，提升 close rate
- 輸出 HTML 可直接部署；輸出 PPTX 可做客戶簡報

### 4.4 太極框架公版 repo landing page（發佈配套）

**現狀**：發佈（04/18 後）需要 public landing page，目前無。

**Claude Design 路徑**：指向 repo codebase → 自動產出符合 repo 風格的 landing page → HTML export → GitHub Pages / Vercel 部署。

### 4.5 共腦系統報告視覺化（內部效率）

**現狀**：週報、月報、季報以純文字為主。growth-history.jsonl、session-state.json 有數據但無圖表。

**Claude Design 路徑**：把 JSON data 餵給 Claude Design → 生成 dashboard 風格簡報 → Owner review 用。

---

## 5. 與太極框架「器可換」原則的張力

Claude Design 是 Anthropic 垂直整合新「器」，但：

- **綁 Anthropic 訂閱**（Pro/Max/Team/Enterprise 才能用）
- **不支援 BYO model**（只吃 Opus 4.7）
- **資料上傳到 Anthropic cloud**（非 self-host）

對照太極框架「器可換」原則，Claude Design 是**典型 vendor lock-in**。

**對應策略**：
- 產出階段可用（加速個人工作）
- 不可成為公版 repo 的必要依賴
- 替代方案儲備：Gamma、Canva Magic Design、Tome、Pitch AI

---

## 6. 需確認事項

1. **Owner 當前 Claude 訂閱等級** — Pro / Max？若無則需評估訂閱成本 vs 節省時間價值
2. **碩論簡報的系所規範** — 是否有字型／配色／排版硬性要求
3. **Passive-income 頻道是否已有 brand design system** — 若無，先用 Claude Design 產一套

---

## 7. 行動建議（優先序）

| 優先 | 行動 | 預期效益 |
|---|---|---|
| P0 | 確認 Claude 訂閱等級 → 若可用，立即跑博班面試簡報 V1 | 4/30 面試準備加速 4–8hr |
| P1 | 對 passive-income 頻道建 brand design system | 日後封面/縮圖批量生產 |
| P2 | 太極框架發佈配套 landing page | 發佈質感提升 |
| P3 | Fay Liu Studio 接案 proposal 模板 | 接案 close rate 提升 |

---

## 8. 貞而無據的部分

- Owner 的 Claude 訂閱等級未確認（memory 無記錄）
- Claude Design 對中文排版品質實測結果未見公開 benchmark
- 與 Canva Pro / Gamma 的功能對比尚無中立第三方評測

---

**參考連結**

- [Anthropic · Introducing Claude Design by Anthropic Labs](https://www.anthropic.com/news/claude-design-anthropic-labs)
- [Claude Help Center · Get started with Claude Design](https://support.claude.com/en/articles/14604416-get-started-with-claude-design)
- [TechCrunch · Anthropic launches Claude Design](https://techcrunch.com/2026/04/17/anthropic-launches-claude-design-a-new-product-for-creating-quick-visuals/)
- [Inc · Saved 10 Hours of Work](https://www.inc.com/ashley-couto/anthropic-claude-design-productivity-saved-time-ai-tools/91333054)
- [Inc · Takes Aim at Figma and Adobe](https://www.inc.com/moses-jeanfrancois/anthropic-takes-aim-at-figma-and-adobe-with-new-claude-design-platform/91332587)
- [iThome 175160 · Anthropic公布設計生成模型Claude Design](https://www.ithome.com.tw/news/175160)
