# PageIndex 深度研究

> 調研日期：2026-05-08
> Repo：https://github.com/VectifyAI/PageIndex
> 本地 clone：/tmp/pageindex（commit @ pushed 2026-05-07）

## 1. 基本資料

| 項目 | 數據 |
|------|------|
| 全名 | VectifyAI/PageIndex |
| 描述 | 📑 PageIndex: Document Index for Vectorless, Reasoning-based RAG |
| License | **MIT**（可商用、可 fork、可修改） |
| 語言 | Python（litellm 1.83.7 / pymupdf 1.26.4 / PyPDF2 3.0.1） |
| Stars | **29,575** |
| Forks | 2,498 |
| Open issues | 142 |
| Subscribers/watchers | 129 |
| 創建日 | 2025-04-01 |
| 最後 push | 2026-05-07（昨天，**極度活躍**） |
| Topics | agentic-ai / rag / reasoning / retrieval / vector-database |
| 商業實體 | VectifyAI（chat.pageindex.ai 提供 SaaS / API / MCP） |

性質：**商業公司的 OSS 引擎 + 雲服務雙軌**。MIT 開源版用標準 PDF parser，雲端版有強化 OCR/tree builder（README 多次提示）。

## 2. 核心功能

### 2.1 一句話定位

把 PDF / Markdown 變成 **語義樹 (table-of-contents tree)**，讓 LLM **不靠 vector similarity** 而靠 **reasoning over tree** 找答案 — 作者稱「Vectorless RAG」。

對標的痛點是 README 開頭那句：「**similarity ≠ relevance**」。長專業文件（金融 / 法規 / 教科書）的 chunk-then-embed RAG 在「相關性」上經常輸給「擬人翻書」式檢索。FinanceBench 號稱 98.7% (README §Introduction)。

### 2.2 演算法（從原始碼讀出來）

主入口 `page_index_main(doc, opt)` (`pageindex/page_index.py:1066`) → `tree_parser` (`page_index.py:1029`) → `meta_processor` (`page_index.py:959`)，分三模式自適應 fallback：

```
check_toc(page_list, opt)  ← 先掃前 toc_check_page_num 頁找目錄頁
   │
   ├── 有 TOC 且帶頁碼 → process_toc_with_page_numbers
   │      1. toc_transformer：LLM 把目錄文字轉 JSON 樹
   │      2. toc_index_extractor：從 toc 後幾頁找 anchor
   │      3. extract_matching_page_pairs + calculate_page_offset：算 logical→physical 偏移
   │      4. process_none_page_numbers：補沒抓到頁碼的節點
   │
   ├── 有 TOC 無頁碼 → process_toc_no_page_numbers (page_index.py:597)
   │      把每頁包 <physical_index_N>...<physical_index_N> tag 餵 LLM 讓它標
   │
   └── 無 TOC → process_no_toc (page_index.py:576)
          直接讓 LLM 從內容自生 TOC（generate_toc_init / generate_toc_continue 連續式）
```

驗證迴路（這是 PageIndex 核心防錯機制，跟一般 RAG 拉開差距的點）：
- `verify_toc` (`page_index.py:900`)：抽 N 個節點，呼 `check_title_appearance` (`page_index.py:13`) 用 LLM fuzzy-match 標題是否真的在該頁出現
- accuracy < 0.6 → fallback 到下一個 mode
- 0.6 < accuracy < 1.0 → `fix_incorrect_toc_with_retries` (`page_index.py:878`) 最多 3 輪修
- accuracy = 1.0 → 通過

樹大時 `process_large_node_recursively` (`page_index.py:1000`)：節點若 > 10 頁且 > 20k tokens 就遞迴拆。

`validate_and_truncate_physical_indices` (`page_index.py:1124`) 把 LLM 幻覺出的越界頁碼砍掉，防 PDF 解析崩潰。

### 2.3 輸入輸出

**輸入**：PDF（PyPDF2 + pymupdf parsing）或 Markdown（按 `#` 階層解析，`page_index_md.py:32`）。

**輸出**：JSON tree，每節點欄位（`run_pageindex.py` + 範例 `examples/documents/results/four-lectures_structure.json`）：

```jsonc
{
  "title": "Financial Stability",
  "node_id": "0006",        // 4-digit zero-padded
  "start_index": 21,         // physical PDF page
  "end_index": 22,
  "summary": "The Federal Reserve ...",   // optional, LLM-generated
  "text": "...",             // optional
  "nodes": [...]             // recursive children
}
```

可選 `doc_description`（全文摘要），整體 wrap 成 `{doc_name, doc_description, structure}`。

### 2.4 Retrieval 三件式（vectorless 核心）

`retrieve.py` 暴露三個函式給 agent 當 tool（`examples/agentic_vectorless_rag_demo.py:62-79`）：

| Tool | 用途 |
|------|------|
| `get_document(doc_id)` | 拿 metadata / page count |
| `get_document_structure(doc_id)` | 拿 tree（去掉 text 欄位省 token） |
| `get_page_content(doc_id, pages="5-7")` | 拉指定頁範圍純文字 |

LLM agent 看 tree → 推理該翻哪幾頁 → 拉那幾頁 → 答。完全沒有 embedding / vector store / cosine similarity。

範例 system prompt (`agentic_vectorless_rag_demo.py:44-52`)：
```
- Call get_document_structure() to identify relevant page ranges.
- Call get_page_content(pages="5-7") with tight ranges; never fetch the whole document.
- Answer based only on tool output.
```

### 2.5 配置（`pageindex/config.yaml`）

```yaml
model: "gpt-4o-2024-11-20"           # tree builder
retrieve_model: "gpt-5.4"            # query agent
toc_check_page_num: 20
max_page_num_each_node: 10
max_token_num_each_node: 20000
```

`# model: "anthropic/claude-sonnet-4-6"` 註解掉但有 — 透過 LiteLLM 支援多 provider，不綁 OpenAI。

### 2.6 成本與限制（誠實面）

- **每節點都呼 LLM**：`toc_transformer` / `verify_toc` / `check_title_appearance_in_start_concurrent` / `generate_summaries_for_structure` 全是 LLM call。一份 100 頁 PDF 索引一次需要數十～百次 GPT-4o 呼叫。**不便宜**。
- **首次索引慢**：相比 vector RAG「一次 embed 之後常數時間檢索」，PageIndex **每次查詢都要 LLM 推理 tree → 拉頁 → 再推理**，retrieval 階段也是 LLM-heavy。
- **依賴 PDF 結構**：scanned PDF / 沒有 TOC 的書效果會掉（會 fallback 到全 LLM 生成，更貴）。
- **Markdown 模式靠 `#` 階層**：README 自承「PDF 轉 markdown 工具大多保不住階層」(README §Markdown support)，所以 md 模式適用範圍窄。

## 3. 對齊判斷

### 3.1 候選評估

| 候選 | 規模 | PageIndex 適配度 | 結論 |
|------|------|------------------|------|
| **reading**（讀書會） | 2 本經典已完成（Epictetus / Zarathustra），未來書單未定 | **低** — 已用「逐章帶讀+討論」模式產出純 markdown 紀錄，沒有「翻長 PDF 找答案」需求；經典原著本身有自然章節，pandoc/手切就夠 | **不採用** |
| **thesis**（論文資料整理） | Paper A/B/C 三篇規劃中，已有 v1.0/v2.0/v1.7/v1.9 多版本 docx + 中興格式 docx + 大量 build_*.py | **中** — 論文寫作主要是「自己產出」不是「檢索」；但 Paper B/C 文獻調研階段需要快速翻 arXiv PDF 摘要 — 可用，但 OpenReview/arxiv-mcp 已有同類工具 | **次選**（孤立用例） |
| **aquafeed**（43 份開發報告） | `archive/AQUAFEED-013_full-system-dev/` 44 個檔案、`docs/` 30+ 份 markdown + PDF（spec / audit / handoff / wiring / production-bom 等） | **高** — 真實的「**長尾文件超過上下文 + 跨文件交叉查問**」場景。譬如「v3 架構提案中如何處理 OTA？」需要同時碰 v3-architecture-proposal.md / ota-esp32-design.md / esp32-ota-guide.md。完美 fit | **首選** |
| **共腦本身**（memory + archive） | memory/ 112 個 .md / 592KB；archive/ 20+ 個 epoch-level snapshot；MEMORY.md 是手寫 index | **中高** — memory 已有 MEMORY.md 一行式 index 機制（與 PageIndex 樹同構但更扁），新增樹索引邊際效益有限；archive/ 是冷儲存、極少查、查時用 git log + grep + Read 已足夠 | **次選**（重疊明顯） |

### 3.2 首選 = aquafeed，理由

1. **資料密度匹配**：43 份報告（INDEX.md 列表）+ docs/ 30 份 = 約 70 份高度交織的工程文件，跨文件查詢頻率高。
2. **已有痛點訊號**：CONTEXT.md 兩個 P1/P2 bug 都是「兩個檔案沒對上」（dependencies.py vs lifespan / api/app.py 沒掛 router）— 正是 PageIndex 樹索引能加速的「跨文件交叉指涉」問題。
3. **未來持續累積**：aquafeed 還在 v2.0，docs/ 會繼續長。memory/ 已穩定，reading 已完成。
4. **產品價值外溢**：aquafeed 是分潤產品（NT$2,500/台×100 台），給代工廠 / 客戶 / 維修工的「文件助理」是實打實的功能（**符合「自由 = 減少對人依賴」太極**）。
5. **規模合理**：70 份 × 平均 5 頁 ≈ 350 頁，索引一次 GPT-4o 估計 USD $5-15 量級，可接受。

### 3.3 爻則 gate 檢查

- 與 contracts/ 矛盾？— `projects/taichi-framework/contracts/` 未讀但本 task 是 research 不動 contracts，N/A。
- 從專案需求推導？— 是。從 aquafeed 已知痛點（跨文件交叉查）+ 太極（減少對人依賴）推導，不是從「PageIndex 很紅所以該用」推導。
- 不可逆？— 否。POC 失敗刪掉就行。

## 4. 整合策略草案

### 4.1 場景定位

**aquafeed-doc-bot**：以 aquafeed 全部 docs/ + archive/AQUAFEED-013 為 corpus，建一棵 PageIndex 樹，給 Owner 查、給未來代工廠 / 客戶 / 維修人員查。

### 4.2 三層架構

```
[Layer 1 索引層]
  cron 或 git post-commit hook
  → run_pageindex.py 對每份 docs/*.md + docs/*.pdf 建樹
  → 落地 results/*.json 進 git（讓 PageIndex 跨裝置同步）

[Layer 2 查詢層]
  PageIndexClient + workspace=aquafeed/.pageindex/
  暴露 3 tool：get_document / get_document_structure / get_page_content
  + 第 4 個 tool：list_documents（跨檔搜尋的入口）

[Layer 3 介面層]
  方案 A：worker 模式 — bash scripts/start-worker.sh aquafeed-doc "問句"
  方案 B：tg-send 模式 — 從 Telegram 問
  方案 C：MCP 模式 — 包成 MCP server 給 Claude Code 直接吃
```

### 4.3 模型選擇（從 config.yaml 修改）

```yaml
# 索引階段（一次性，可慢可貴）
model: "anthropic/claude-sonnet-4-6"     # tree builder（已在 config 註解中提示）

# 檢索階段（每次查詢，要快）
retrieve_model: "anthropic/claude-haiku-4-5-20251001"
# 等等 — feedback_no_haiku.md 寫「永遠不使用 haiku」⚠️
# → 改用 sonnet-4-6 雙線。
```

**[未驗證]** 雙 sonnet 是否會讓查詢過慢，需要 POC 後實測。

### 4.4 與共腦既有機制的整合

| 共腦機制 | PageIndex 對應 | 動作 |
|----------|----------------|------|
| MEMORY.md 一行式 index | 平面，PageIndex 是樹 | **不取代**。MEMORY.md 規模 < 200 行的甜蜜點 PageIndex 不划算 |
| `lesson-scan.sh` | grep-based | **不取代**。lessons 數百行 markdown，grep 已足夠 |
| `start-worker.sh "查 aquafeed X"` | 派手翻文件 | **可取代**。worker 翻 Read+grep 慢且 token 貴，PageIndex 做主索引省 token |
| `feedback_factcheck_before_deliver.md` | 數值/引用查核 | **互補**。PageIndex 找到頁 → 仍要 Read 原檔 ground truth |

### 4.5 風險

1. **依賴 PDF 結構而非語義內容**：aquafeed docs/ 大多是 markdown，PageIndex 的 md 模式 README 自承「保不住階層」。需先 POC 確認對「自己寫的、有清楚 # 階層」的 markdown 是否 OK。
2. **冷啟動成本**：70 份文件首次索引估 $5-15 USD，rate limit 7-day 已逼近 92% 警戒線（CLAUDE.md 啟動校準第 6 條）。
3. **過度工程**：Owner 自己一個人查 aquafeed docs，grep + Read 也行。**價值門檻是「外人查」（代工廠 / 客戶）**，現在還沒有外人。先做 POC 不全鋪。
4. **更新成本**：docs 一改就要重建那份子樹 — `client.index()` 是一次性的，無 incremental update。

## 5. 立即可動手的 next 3 actions

### Action 1：POC — 對 aquafeed 兩份典型文件試索引（**P0，1-2 小時**）

```bash
cd /tmp/pageindex
pip3 install -r requirements.txt
export OPENAI_API_KEY=$(cat ~/.secrets/openai-key)  # 待確認 key 是否在 secrets
# 試 PDF
python3 run_pageindex.py --pdf_path /home/ra/ai-group/projects/aquafeed/docs/wiring-assembly-2026-04-24.pdf \
  --model "anthropic/claude-sonnet-4-6"
# 試 markdown
python3 run_pageindex.py --md_path /home/ra/ai-group/projects/aquafeed/docs/v3-architecture-proposal.md \
  --model "anthropic/claude-sonnet-4-6"
```

驗收：
- [ ] PDF 結果 tree 對得上肉眼讀的章節層級
- [ ] markdown 結果 line_num 正確，能用 get_page_content 拉回原文
- [ ] 估算單份索引成本（token 數 × pricing）

### Action 2：整合測試 — agent 端到端問答（**P1，2-3 小時，POC 通過後再做**）

跑 `examples/agentic_vectorless_rag_demo.py` 改 corpus 為 aquafeed 3-5 份代表文件，測：
- [ ] 「v3 架構如何處理 OTA？」（跨 v3-architecture-proposal.md / ota-esp32-design.md / esp32-ota-guide.md）
- [ ] 「ESP32 wiring 接線確認的步驟在哪？」（找 wiring-verify-2026-04-24.md）
- [ ] 「production BOM 上 ESP32 的型號？」（找 production-bom.md）

對照組：同樣三題用 grep + Read 由 worker 跑，比較 token 數 / 時間 / 答案正確性。

### Action 3：寫進 CONTEXT.md + LANDSCAPE.md（**P2，30 分鐘**）

POC 結論寫回兩處：
- `projects/taichi-framework/research/LANDSCAPE.md` — 加 PageIndex 段，比較 vs khazix-skills 的 hv-analysis（後者也是 PDF→深度研究，但 output 是 PDF 報告，PageIndex output 是 tree index — 不同層）
- `projects/aquafeed/CONTEXT.md` — 若 POC 成功加「文件查詢工具：PageIndex 索引在 .pageindex/，用 `bash scripts/aquafeed-ask.sh "問句"`」

## 6. 不確定 / 待驗證標記

- **[未驗證]** OpenAI API key 是否落地 RPi5 ~/.secrets/。memory/infra_api_keys.md 寫「Kaggle + Roboflow 已落地」沒提 OpenAI。POC 前需 Owner 確認或用 ANTHROPIC_API_KEY + LiteLLM 路由。
- **[未驗證]** sonnet-4-6 對 PageIndex prompt 格式（要求嚴格 JSON 輸出）的相容性 — config.yaml 預設仍是 gpt-4o，註解中的 sonnet 是 hint 不是已驗證路徑。
- **[未驗證]** markdown 模式對 aquafeed 自寫 markdown 的階層保留率 — README 警告針對「PDF→md 轉換器產出」，自寫應該 OK 但需測。
- **[未驗證]** PageIndex 對中英混合文件（aquafeed docs/ 大量中文）的 LLM prompt 表現 — 原始 prompts 全英文（page_index.py:23-37 等），對中文 fuzzy match 可能有 corner case。

## 7. 一句話結論

PageIndex 是「**reasoning-based RAG over hierarchical doc tree**」的高品質開源 + 商業實作（MIT、29.5k stars、活躍）。對共腦最有價值的應用點不是 memory/archive（已有 MEMORY.md 機制），而是 **aquafeed 跨文件查詢**（外人友善 + 產品外溢）。建議按上面 Action 1-3 跑 POC 再決定是否常駐。
