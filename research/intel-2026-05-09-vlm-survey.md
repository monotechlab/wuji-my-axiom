# Intel 2026-05-09: 2025-2026 主流 VLM 對全 project leverage 深度研究

> Vision Language Model 全景掃描 + 對共腦 + 對 Owner 全 project 的 leverage 矩陣

查證日期：2026-05-09
來源：各模型官方 repo / paper / model card（詳見「來源清單」）

---

## 1. 基本資料 + 事實接地

### 1.1 雲端 API（旗艦多模態）

| 模型 | 廠商 | 發表 | 參數量 | Context | MMMU | 圖像輸入 | 商用授權 | 來源 |
|---|---|---|---|---|---|---|---|---|
| **GPT-5** | OpenAI | 2025-08 | 未公開 | 256K+ | **84.2%** | 起始 detail level；GPT-5.4 起單張 10.24M px / 6000px max | API 商用 | OpenAI 官方 |
| **Claude Sonnet 4** | Anthropic | 2025-05 | 未公開 | 200K | 74.4% | 多張、文件 PDF native | API 商用 | Anthropic |
| **Claude Sonnet 4.5** | Anthropic | 2025-09-29 | 未公開 | 200K | [未驗證] 應 ≥4.0 | 同上 | API 商用 | Anthropic |
| **Gemini 2.5 Pro** | Google | 2025-Q1 | 未公開 | 2M（業界最長） | **81.7%** | 圖+影片+音訊原生 | API 商用 | Google |
| **Pixtral 12B** | Mistral | 2024-09 | 12B + 400M vision encoder | 128K | 62.5% | 原生解析度 + 變動長寬比 | **Apache 2.0**（OSS+API） | arXiv 2410.07073 |

**關鍵差異**：Gemini 2.5 Pro context 2M 是唯一能整段消化長影片；GPT-5 MMMU 第一；Claude 強在 agentic 工具用，純 vision benchmark 不是 SOTA；Pixtral 是商用級 OSS。

### 1.2 開源大型（自部署旗艦）

| 模型 | 機構 | 發表 | 參數量 | Context | MMMU | License | 來源 |
|---|---|---|---|---|---|---|---|
| **Qwen2.5-VL-72B** | 阿里巴巴 | 2025-01-26 | 72B | 32K（可擴 128K） | **70.2%** | Qwen LICENSE（商用免費，>1 億 MAU 需授權） | HuggingFace |
| **Qwen2.5-VL-7B** | 阿里巴巴 | 2025-01-26 | 7B | 32K | ~58% [未驗證精確] | 同上 | 同上 |
| **InternVL3-78B** | 上海 AI Lab | 2025-04-11 | 78B | 32K | **72.2%**（開源 SOTA） | MIT | arXiv 2504.10479 |
| **InternVL3.5** | 上海 AI Lab | 2025-08-26 | 8B-78B | 同上 | [未驗證] 應 >3.0 | MIT | internvl.github.io |
| **Llama 3.2 Vision 90B** | Meta | 2024-09 | 90B | 128K | [未驗證] 60+ 區間 | Llama 3.2 license（商用，受 Meta 條款限制） | Meta blog |
| **Llama 3.2 Vision 11B** | Meta | 2024-09 | 11B | 128K | ~50% [未驗證精確] | 同上 | 同上 |
| **DeepSeek-VL2** | DeepSeek | 2024-12 | 27.5B MoE / 4.2B 啟用 | 4K [未驗證] | **72.2%** | DeepSeek License（商用） | github DeepSeek-VL2 |
| **DeepSeek-VL2-small** | DeepSeek | 2024-12 | 16.1B MoE / 2.4B 啟用 | 同上 | ~70 [未驗證精確] | 同上 | 同上 |

**MoE 紅利**：DeepSeek-VL2 4.2B 啟用打贏 GPT-4o（70.7），是 2024-Q4 最大開源驚喜。InternVL3-78B 開源 SOTA。Qwen2.5-VL 是部署最成熟、ecosystem 最完整。

### 1.3 邊緣可部署（消費級 GPU / RPi5 / 手機）

| 模型 | 機構 | 發表 | 參數量 | Context | MMMU/OpenCompass | VRAM | License | 來源 |
|---|---|---|---|---|---|---|---|---|
| **MiniCPM-V 2.6** | 清華 OpenBMB | 2024-08 | 8B（SigLip-400M + Qwen2-7B） | 32K | OpenCompass 65.2 | ~6 GB int4 | OSS 商用 | HF openbmb/MiniCPM-V-2_6 |
| **MiniCPM-o 2.6** | 同上 | 2025 | 8B | 同上 | ~70 [未驗證精確] | 同上 | 同上 | HF MiniCPM-o |
| **Phi-3.5 Vision** | Microsoft | 2024-08 | 4.2B | 128K | [未驗證精確]（強於 Claude 3 Haiku） | **3.2 GB** | MIT | HF microsoft/Phi-3.5-vision-instruct |
| **Moondream2** | Vikhyat / m87-labs | 持續更新（2025-06-21 last） | 1.86B（SigLIP+Phi-1.5） | 短 | OCR/UI/小物件強 | <2 GB | Apache 2.0 | HF vikhyatk/moondream2 |
| **Moondream 0.5B** | 同上 | 2025 | 500M | 短 | 邊緣優化 | <1 GB | Apache 2.0 | moondream.ai |
| **SmolVLM 2B** | HuggingFace TB | 2024-11 | 2B | **16K**（RoPE 273K base） | 中段 | <4 GB | Apache 2.0 | HF SmolVLM-Instruct |
| **SmolVLM 500M / 256M** | HuggingFace TB | 2025 | 500M / 256M | 16K | 低（極致小） | <1 GB | Apache 2.0 | HF SmolVLM 系列 |

**邊緣甜點**：MiniCPM-V 2.6 是 8B 級 SOTA（單圖打贏 GPT-4o-mini / GPT-4V / Gemini 1.5 Pro / Claude 3.5 Sonnet）；Phi-3.5 Vision 3.2GB VRAM 是筆電/桌機跑得動的最低成本；Moondream2 / SmolVLM 是 RPi5 / 手機可跑的真小模型。

### 1.4 物理 AI / 具身（VLA, Vision-Language-Action）

| 模型 | 機構 | 發表 | 參數量 | 用途 | 部署 | License | 來源 |
|---|---|---|---|---|---|---|---|
| **Cosmos-Reason1-7B** | NVIDIA | 2025-03-18（GTC）/ 2025-05-17（HF 釋出） | 7B | 物理常識推理、長 CoT、規劃 | 雲/邊（Jetson 級） | OSS（NVIDIA Open Model License） | github nvidia-cosmos/cosmos-reason1 |
| **Cosmos-Reason1-56B** | NVIDIA | 同上 | 56B | 同上，更強 | 雲為主 | 同上 | 同上 |
| **OpenVLA** | Stanford / TRI | 2024-06；FAST tokenizer 2025-01 | 7B（Llama2 + DINOv2 + SigLIP） | 機器人操控 | 含 ADP 1.35× 加速 | OSS | arXiv 2406.09246 |
| **RT-2-X** | Google DeepMind | 2023-10 | 55B | 跨 embodiment 操控 | 雲端為主 | 閉源 | DeepMind |
| **Helix** | Figure AI | 2025-02 | 7B（System 2）+ 80M（System 1） | 雙臂、35 DoF、200Hz 低層控制 | **嵌入式 GPU 機上** | **閉源** | figure.ai/news/helix |

**關鍵分野**：Cosmos-Reason1 是 NVIDIA 推給整個機器人業界的「物理 CoT 大腦」（1X / Agility / Figure / Skild AI / Uber 等都用），OSS。OpenVLA 是學術 SOTA 開源。Helix 是首個全機上跑的雙腦結構（7B 規劃 + 80M 控制），但閉源。RT-2 雖是先驅但 55B 雲端為主、已被 OpenVLA 7B 超越 16.5%。

### 1.5 對比矩陣速查（leverage 用）

| 屬性 | 雲端旗艦 | 開源大型 | 邊緣 | 具身 VLA |
|---|---|---|---|---|
| 部署成本 | $$$/call | 1×A100/H100 起 | RPi5 / 筆電可跑 | Jetson / 機上 |
| MMMU 80%+ | GPT-5/Gemini2.5Pro | 0 | 0 | 不適用 |
| MMMU 70%+ | Claude4 / GPT-5 | InternVL3, DeepSeek-VL2, Qwen2.5-VL-72B | 0 | 不適用 |
| 隱私（本地） | ❌ | ✅ | ✅ | ✅ |
| 即時控制 | ❌ | ❌ | △（小模型可） | ✅（Helix 200Hz） |
| 商用 OSS | Pixtral | Qwen / InternVL / DeepSeek | MiniCPM-V / Phi3.5V / Moondream / SmolVLM | OpenVLA / Cosmos-Reason |

---

## 2. 對共腦的 leverage（4 切點判斷）

### 切點 A：spawn worker 用 VLM 做圖像理解任務？✅ 適用（高）

**現狀差距**：共腦 worker 全是純文字（claude CLI tmux），所有「需要看圖」的任務（截圖驗證、PCB 照片、AOI 樣本、Owner 貼的論文截圖、影片關鍵幀）都需 Owner 人眼介入。

**可動手 use case**：
1. **worker 截圖自驗**：worker 完成 GUI 操作後（如 PowerPoint 自動化、Pro Tools 編輯）截圖丟 VLM 確認結果是否合理 → 取代 Owner 抽查
2. **Owner 貼圖預處理**：Owner 截圖（簡報、論文 figure、儀表照）先給 VLM 萃取結構化文字 → 再進共腦推理鏈（解 Owner 「視覺產出必須驗證」memory 的瓶頸）
3. **OCR + 結構萃取**：表格、PCB silkscreen、儀表板 → 不再依賴 PaddleOCR（與 04-30 lesson「百度系紅線」相容）

**選型建議**：
- **本地 worker 預設**：MiniCPM-V 2.6（8B int4 ~6GB，筆電 4070 跑得動，OSS 商用，OCR/UI 強）
- **Owner 貼圖即時校驗**：Claude Sonnet 4.5（已在 session 內，最低摩擦，不需另外 spawn）
- **批次離線**：InternVL3-8B（MIT，開源 SOTA，無 Qwen >1億 MAU 條款風險）

### 切點 B：memory 系統收 VLM benchmark 速查表？✅ 適用（中）

當前 memory 只有 `infra_claude_subscription.md` 提 Max plan，無 VLM 規格。建議補：
- `reference_vlm_2026.md` — 本表壓縮版（5 個關鍵 row × 4 個必要欄位），用於 worker dispatch 時快速選型
- 觸發條件：每季更新一次（VLM 季度大改版）

### 切點 C：daily-intel xray-skills 加 VLM 關鍵字？✅ 適用（高）

**現狀**：05-07 xray 把 Cosmos / OpenVLA / Audio-Visual 都丟「store, over budget」。代表掃描器**沒有 VLM 獨立分類**，全靠 cs.RO/cs.CV 兜底。

**動作**：
- 在 `infra/xray-keywords.json`（或對應設定）加：`vision-language model` / `multimodal LLM` / `MLLM` / `VLA` / `vision-language-action` / `embodied AI` / `physical AI` / `world model`
- 路由規則：VLM 命中 → taichi-framework（架構參考）+ 對應應用 project（aidc-aoi/cucumber/usv 等）雙投

### 切點 D：skill metabolism — VLM tool use 範式 vs 共腦 worker dispatch？✅ 適用（中）

**結構同構觀察**：VLM 的「視覺 tool use」（CLIP / SAM / detection model 串接 LLM 規劃）與共腦「worker dispatch」是**同類控制問題**：
- VLM 業界已收斂出 ReAct-style「規劃 → 工具調用 → 觀察 → 再規劃」迴圈
- Helix 雙腦結構（System 2 規劃 + System 1 執行）= 共腦「Mono 規劃 + worker 執行」**完美同構**

**反向收穫**：Helix System 2 / System 1 的**頻率比 7-9Hz : 200Hz ≈ 1:25** 是個強訊號——共腦目前 Mono 對 worker 的「規劃頻率 vs 執行頻率」比沒有刻意設計，可能規劃過多、執行不夠。建議在 calibration 加一條「Mono 一次 plan 應對應 ≥10× worker 動作」。

---

## 3. 對 Owner 各 project 的 leverage 矩陣

| Project | Leverage | 推薦模型 | 切入點 | 優先級 |
|---|---|---|---|---|
| **aidc-aoi** | **極高** | InternVL3-8B 本地 + Cosmos-Reason1-7B 物理 CoT | VLM 取代 PatMax 人工閾值（對應 5/7 xray Task-Aware Scanning 論文）；缺陷描述+判決一體 | **P0** |
| **cucumber-detection** | **高** | Moondream2 / Qwen2.5-VL-7B fine-tune | CLIP 零樣本失敗（已封存負結果），改走 VLM **few-shot prompted detection** + 弱監督 | **P0** |
| **usv** | **高** | Cosmos-Reason1-7B 雲端規劃 + OpenVLA 思維借鏡 | 高層任務分解（讀對應 5/7 xray "Say the Mission" 論文路線）；不直接跑機上 | P1 |
| **agv** | 高 | Cosmos-Reason1-56B 規劃 + 小 VLM 機上 | RMFS 多車調度 + 視覺場景理解（5/7 xray SOAR 論文相關） | P1 |
| **scal3r-product** | 高 | InternVL3 / Gemini 2.5 Pro 影片 | survey video → 語意建圖（FUS3DMaps 路線）；2M context 整段消化 | P1 |
| **passive-income** | **高** | GPT-5 / Claude 4.5 Sonnet vision + Pixtral | 影片素材篩選（從 Pexels 下載先 VLM 評分）、自動字幕 timing 校對、生成稿 vs 視覺一致性檢查 | P0 |
| **soramimi** | 中-高 | Qwen2.5-VL-7B（聲音譜+歌詞圖） | 音頻 Mel spectrogram 視覺化後丟 VLM 找空耳對齊（規避音頻 foundation model 缺口） | P2 |
| **pid-solver** | 中-高 | Cosmos-Reason1-7B | 「貼車輛照片+spec → 推薦初值」是天然 VLM 切點，與 SaaS 化路線契合 | P1 |
| **muse-spark-research** | 中 | Phi-3.5 Vision 邊緣 | EEG 視覺化（topographic map）→ VLM 即時口語描述，Owner 可邊看邊聽 | P2 |
| **pet-id** | 中 | Moondream2 0.5B | 寵物特徵描述（毛色/紋路）作為 ID embedding 的補充信號 | P2 |
| **fan-panel** | 中 | MiniCPM-V 2.6 機上 | 風扇面板 OCR + 故障燈號讀取，本地化（工廠內網） | P2 |
| **aidc-notebook** | 中 | 同 aidc-aoi | 筆記本同 AOI 路線，但任務以教學/sandbox 為主，VLM 是現成示範 | P2 |
| **aquafeed** | 中 | Claude 4.5 vision | 70+ 報告含表格/圖表，雲端 VLM 抽結構化內容（已驗證 Adobe automation 路線，補上 vision） | P2 |
| **drink-station** | 低-中 | MiniCPM-V 2.6 RPi5 | 點餐 UI / 庫存照片識別，但飲料站零預算策略應後置 | P3 |
| **kr6** | 低-中 | Phi-3.5 Vision | 機器人視覺輔助，但 kr6 是 stale 狀態 | P3 |
| **omniverse** | 低-中 | Cosmos-Reason1（同源 NVIDIA 生態） | Isaac Sim + Cosmos 物理 CoT 是 NVIDIA 官方路線；但 omniverse 非當前焦點 | P3 |
| **local-distillation** | **極高（meta）** | 蒸餾目標本身是 VLM | Qwen2.5-VL-7B / Moondream2 是「Claude 蒸餾本地備援」memory 的具體實作目標 | P0 |
| **thesis** | 中 | Claude 4.5 vision | 圖表審稿、reviewer 截圖回覆 | P2 |
| **reading** | 低 | Phi-3.5 Vision | 書籍封面/章節結構識別，興趣驅動，無壓力 | P3 |
| **platycerium** | 低 | Moondream2 | 鹿角蕨健康狀態識別（葉色/孢子囊），興趣專案 | P3 |
| **taichi-framework** | 高（meta） | 全部 | 本研究本身即 leverage；治理框架需把 VLM 列為一等公民 | P0 |
| **passive-income / Phase 3** | 高 | GPT-5 vision + Adobe MCP | 配合 04-28 connector lesson，VLM 看 After Effects/PR 預覽截圖→自動微調 | P1 |
| **infra-security** | 中 | Phi-3.5 Vision 本地 | 監視畫面 / log 異常截圖識別 | P2 |
| **esp-blast** | 低 | Moondream2 0.5B（極輕） | 嵌入式硬體照片識別（焊接、引腳），現場除錯助手 | P3 |
| **huashu-integration / cardvortex / haoshi-accelerator / tm12 / open-design-research / geo_nodes** | 低 | — | 大多 stale 或無視覺需求 | P3 |

**最高 ROI 三配對**（P0）：
1. **aidc-aoi × InternVL3-8B 本地** — 對應已知用例（PatMax 替代）+ 已有客戶/供應鏈+ Cosmos-Reason1 物理 CoT 加分
2. **passive-income × GPT-5/Claude 4.5 vision** — 已建 pipeline，VLM 是品質飛輪（素材篩選+一致性檢查）
3. **local-distillation × Qwen2.5-VL-7B / Moondream2** — 把「Claude 蒸餾備援」記憶具體化，OSS 路線清晰

---

## 4. 全面風險評估（5 類）

### 4.1 技術風險：MEDIUM
- **MMMU 飽和**：Gemini 2.5 Pro 81.7% / GPT-5 84.2% 已逼近人類專家上限，benchmark 信號正在失效——選型不能只看分數
- **VLM 幻覺特化問題**：「看見不存在的東西」比純 LLM 更隱蔽（OCR 數字錯一位、判斷有/無物件相反）；aidc-aoi 工業應用必須有獨立驗證層
- **小模型解析度上限**：MiniCPM/Phi/Moondream 對 PCB 細節（μm 級）可能無解，需要與傳統 CV 串接
- **控制措施**：所有 VLM 輸出必過「ground truth 驗證」（呼應 `feedback_trust_breach.md`），絕不單點交付

### 4.2 商業授權風險：MEDIUM
- **Qwen License**：>1 億 MAU 需另談——對 passive-income SaaS 化是潛在天花板，但目前無壓力
- **Llama 3.2 條款**：Meta 條款限制 70 億 MAU 以上應用，且要顯示「Built with Llama」
- **DeepSeek License**：較寬鬆但中國公司——某些客戶（aidc-aoi 半導體？）可能列禁
- **Helix 閉源**：學不到，只能參考結構
- **控制措施**：商用 path 預設 Apache 2.0（Pixtral / Moondream / SmolVLM）或 MIT（InternVL3）；Qwen 留做「dev/non-prod」；嚴格客戶需求改 InternVL3-78B

### 4.3 路徑依賴鎖定風險：HIGH
- **HF 生態鎖定**：模型發行 99% 在 HF，但 HF 商用條款近年變動（私有 inference 端點計費調整）
- **NVIDIA 鎖定**：Cosmos-Reason 套件需 CUDA + Triton，不易移植到非 NVIDIA 邊緣（呼應 04-30 Mythos 教訓）
- **API 漲價**：GPT-5 / Claude / Gemini 都歷史性漲過價（雖然多在 token 維度而非 vision 額外計費）
- **控制措施**：本地 OSS 版本必須**先驗證可跑**再依賴雲端（與 `feedback_wait_distilled_or_better_hw.md` 一致——VRAM > 1.5× 等等再說）；雲端 API 包一層 abstraction（litellm / aisuite）防綁死

### 4.4 安全 / 隱私：MEDIUM
- **截圖含敏感資料**：Owner 截圖可能含 email、API key、客戶資料——丟雲端 VLM 等於外流
- **prompt injection via image**：圖片內藏指令 → VLM 跟著做（已有公開 attack）
- **控制措施**：截圖類任務預設本地 VLM（MiniCPM-V 2.6 / Phi-3.5 V）；雲端只用於 Owner 已知公開內容（論文 figure / 公開新聞圖）

### 4.5 認知 / 流程風險：LOW
- **Owner 對 VLM 過度信任**：「VLM 看了說沒問題」可能變成 lazy validation（已有 `feedback_verify_visual_output.md` 防線）
- **控制措施**：VLM 結論必須 cross-check（看不同模型、或 VLM + 程式邏輯雙路）；不接受 VLM 為唯一通過閘

---

## 5. 自洽校核（vs Owner memory）

| Memory 條目 | 與本研究的關係 | 結論 |
|---|---|---|
| `feedback_no_haiku.md`（永遠不用 haiku） | 雲端選型不選 Claude Haiku Vision | ✅ 已遵守，本研究選 Sonnet 4.5 / GPT-5 / Gemini 2.5 Pro 不選 Haiku |
| `infra_claude_subscription.md`（Max plan） | Claude vision 已包含 | ✅ Owner 貼圖路線零邊際成本 |
| `project_local_distillation.md`（蒸餾本地備援） | VLM 蒸餾是同路線延伸 | ✅ 三 P0 之一 |
| `feedback_wait_distilled_or_better_hw.md`（VRAM > 1.5× 等等） | InternVL3-78B 在 4070 上不可行（需 H100） | ✅ 推 InternVL3-8B 不推 78B |
| `feedback_verify_visual_output.md`（視覺產出必驗證） | VLM 自驗 worker 截圖正是此 leverage | ✅ 切點 A use case 1 直接對應 |
| `feedback_trust_breach.md`（必須 ground truth 驗證） | VLM 幻覺須獨立驗證 | ✅ 風險 4.1 控制措施已含 |
| `infra_dataset_cache.md`（中央快取） | 大量 VLM 預訓練/微調資料應入 /home/ra/datasets-cache | ✅ 後續實作須遵循 |
| `feedback_template_inertia.md`（識別模板慣性） | 不應把 LLM 模板照搬 VLM | ✅ 已分開處理 prompt-with-image 範式 |
| `project_passive_income_allgen.md`（純 AI 全生成） | VLM 校驗一致性是品質飛輪 | ✅ 切點清晰 |
| 04-30 「百度系紅線」（PaddleOCR 不採用） | OCR 改走 VLM 路線 | ✅ aidc-aoi/aidc-notebook 切點本來就是 VLM 取代 OCR |

**衝突點**：無。

---

## 6. 立即可動手 actions（低承諾、可逆、不付費）

### 本週可做（A1-A5）
1. **A1 — xray 補關鍵字**：infra/xray 設定加 VLM/MLLM/VLA/embodied/physical AI/world model 6 個關鍵字（Effort: 30min, Reversible: 100%）
2. **A2 — memory 補速查表**：寫 `memory/reference_vlm_2026.md`（5 row × 5 col 壓縮版）（30min, Reversible）
3. **A3 — RPi5 跑 Moondream2 0.5B**：ollama pull 一個最小 VLM，驗證共腦本機可呼叫（1hr, Reversible）
4. **A4 — 筆電跑 MiniCPM-V 2.6 int4**：4070 12GB 應夠，benchmark 一張 aidc-aoi 樣本 vs 一張 cucumber 樣本，看實際表現（2hr, Reversible）
5. **A5 — 給共腦加 vlm-validate-screenshot.sh**：包一層 bash 接受 image path → 呼叫本地 VLM → 回 「pass / fail / 描述」（1hr, Reversible）

### 兩週內（B1-B3，需小研究）
1. **B1 — aidc-aoi VLM PoC**：在現有資料集挑 10 張，跑 InternVL3-8B 對比 PatMax 人工閾值，產出 `aidc-aoi/research/intel-vlm-poc.md`
2. **B2 — passive-income 影片素材篩選器**：給 Pexels 下載片段+腳本一句話，VLM 打 0-10 分相關性，串到既有 pipeline
3. **B3 — local-distillation TODO 更新**：把「VLM 蒸餾」具體化為三 candidate（Qwen2.5-VL-7B / Moondream2 / Phi-3.5V）的對比實驗計劃

### 不做（明確排除）
- ❌ 自己訓練 VLM（成本巨大，不在第一性原理）
- ❌ 部署 InternVL3-78B（VRAM 超規格 1.5×，呼應 lesson）
- ❌ 採用 Helix 思路強行雙腦（共腦目前單一 Mono 是刻意設計，過早分層是過度工程）
- ❌ 對所有 project 全面導入（焦點不是普攻，P0 三項先打通）

---

## 7. 結論（含 But-For 推導）

### 7.1 But-For 推導
- **若無 VLM**：共腦完全失明，所有視覺任務必須 Owner 親自處理 → 每週估計 ≥3hr 視覺 toil（截圖貼 prompt + 人工驗證 + OCR 校對）
- **有了 VLM 但部署在雲**：隱私+成本+網路依賴 → Owner 「在哪共腦在哪」哲學受損
- **本地 VLM 上 RPi5 / 4070**：消除 toil + 保留隱私 + 零邊際成本 → 直接擴大共腦能力邊界
- **結論**：本地 VLM 是 P0 必做，不是 nice-to-have

### 7.2 復發測試
- 若延後 6 個月不導入 VLM：aidc-aoi 仍卡 PatMax 工人工閾值（無法量產）；passive-income 品控仍靠 Owner 抽查（限速）；cucumber 已封存負結果無翻盤路徑
- 反向：若立即導入但不做風險 4.1 的獨立驗證 → 第一個 false positive 上線即信任崩盤（呼應 trust_breach memory）
- → **必做但須帶獨立驗證層**

### 7.3 解釋測試（為何現在做？）
- **Q1**：為何不是 2025 年初？— 因 Qwen2.5-VL（01）、InternVL3（04）、Cosmos-Reason1（03）、SmolVLM 系列（25 全年迭代）、MiniCPM-V 2.6 8B 級 SOTA — 全在 2025 年成熟
- **Q2**：為何不等 2027？— GPT-5 已 84.2% MMMU，benchmark 飽和；2026 主要是「邊緣化 + 工業化」（Helix 機上、Cosmos 物理 CoT），而非「再升一個量級」——拐點已過
- **Q3**：為何此刻發起？— 與 04-28 Adobe connector + 05-07 xray Task-Aware Scanning 論文構成三角訊號，外部生態 + 工業需求 + Owner 焦點專案（thesis/usv 收尾後將進入 aidc/passive-income 強化期）三邊對齊

### 7.4 終局判斷
**做。三 P0 並行（aidc-aoi PoC / passive-income 篩選器 / local-distillation 蒸餾候選）。本週先打 A1-A5 五個低承諾動作驗證可行性，兩週內 B1-B3 收斂出對 aidc-aoi 的具體導入方案。**

---

## 來源清單

### 雲端 API
- OpenAI GPT-5 — https://openai.com/index/introducing-gpt-5/
- OpenAI GPT-5.4（image detail level）— https://openai.com/index/introducing-gpt-5-4/
- Anthropic Claude — https://www.anthropic.com/claude
- Google Gemini 2.5 Pro — https://ai.google.dev/gemini-api/docs
- Mistral Pixtral 12B — https://arxiv.org/abs/2410.07073 ; https://huggingface.co/mistralai/Pixtral-12B-2409

### 開源大型
- Qwen2.5-VL-72B — https://huggingface.co/Qwen/Qwen2.5-VL-72B-Instruct
- InternVL3 — https://arxiv.org/abs/2504.10479 ; https://internvl.github.io/blog/2025-04-11-InternVL-3.0/
- InternVL3.5 — https://internvl.github.io/blog/2025-08-26-InternVL-3.5/
- Llama 3.2 Vision — https://huggingface.co/meta-llama/Llama-3.2-90B-Vision-Instruct ; https://ai.meta.com/blog/llama-3-2-connect-2024-vision-edge-mobile-devices/
- DeepSeek-VL2 — https://github.com/deepseek-ai/DeepSeek-VL2

### 邊緣
- MiniCPM-V 2.6 — https://huggingface.co/openbmb/MiniCPM-V-2_6
- MiniCPM-o 2.6 — https://huggingface.co/openbmb/MiniCPM-o-2_6
- Phi-3.5 Vision — https://huggingface.co/microsoft/Phi-3.5-vision-instruct
- Moondream2 — https://huggingface.co/vikhyatk/moondream2 ; https://moondream.ai/
- SmolVLM — https://huggingface.co/blog/smolvlm ; https://huggingface.co/HuggingFaceTB/SmolVLM-Instruct

### 物理 AI / 具身
- Cosmos-Reason1 — https://github.com/nvidia-cosmos/cosmos-reason1 ; https://docs.nvidia.com/cosmos/latest/reason1/index.html
- Cosmos paper — https://research.nvidia.com/publication/2025-03_cosmos-reason-1-physical-ai-common-sense-embodied-decisions
- OpenVLA — https://arxiv.org/abs/2406.09246 ; https://github.com/openvla/openvla
- Helix (Figure AI) — https://www.figure.ai/news/helix
- RT-2 — https://robotics-transformer2.github.io/

### Benchmark
- MMMU — https://mmmu-benchmark.github.io/
- MMBench Leaderboard — https://mmbench.opencompass.org.cn/leaderboard
- MMMU Pro / vals.ai — https://www.vals.ai/benchmarks/mmmu

### 內部相關
- 共腦 memory — `~/.claude/projects/-home-ra-ai-group/memory/MEMORY.md`
- 5/7 xray VLM 相關 — `infra/xray/2026-05-07.md`（Task-Aware Scanning / FUS3DMaps / Audio-Visual Intelligence / Cosmos 鏈條）
- 04-30 PaddleOCR 拒絕 lesson — commit 486e21e6
