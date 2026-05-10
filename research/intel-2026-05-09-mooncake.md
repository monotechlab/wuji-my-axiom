# Mooncake (kvcache-ai/Mooncake) — Intel X-collision 深度研究

> 5/9 追溯補派（worker dispatched 2026-05-10）。對象：xray 5/9 W-wildcard 名單之一。

## 1. 基本資料 + 事實接地

| 項目 | 值 | 出處（查證日 2026-05-10） |
|---|---|---|
| URL | https://github.com/kvcache-ai/Mooncake | xray seed |
| 性質 | LLM 推論服務平台（disaggregated KVCache architecture，C++ 為主） | repo description |
| 隸屬 | kvcache-ai org，Kimi (Moonshot AI 北京月之暗面) 線上服務的開源版 | repo README |
| Stars | 5288 | `gh api repos/kvcache-ai/Mooncake` 2026-05-10 |
| Forks | 729 | 同上 |
| Watchers / Subscribers | 5288 / 43 | 同上 |
| Open issues | 428 | 同上 |
| Created / Last push | 2024-06-25 / 2026-05-09 | 同上（極度活躍，當天還在推 commit） |
| License | Apache-2.0 | 同上 |
| Topics | disaggregation, inference, kvcache, llm, rdma, sglang, vllm | 同上 |
| 主頁 | https://kvcache-ai.github.io/Mooncake/ | repo `homepage` 欄位 |
| 學術背書 | FAST'25 Best Paper（Mooncake: A KVCache-centric Disaggregated Architecture for LLM Serving）+ arXiv 2407.00079 | https://www.usenix.org/system/files/fast25-qin.pdf, https://arxiv.org/abs/2407.00079 |
| 作者 | Ruoyu Qin, Zheming Li, Weiran He, Mingxing Zhang, Yongwei Wu, Weimin Zheng, Xinran Xu | arxiv 2407.00079（清華系統組 + Moonshot 工程團隊，校企混合） |

### 架構（README + arxiv 摘要交叉印證）

- **KVCache-centric disaggregated**：把 prefill 與 decode 拆成兩個 cluster，共享一個橫跨 GPU/CPU/DRAM/SSD 的 KVCache 池
- **Transfer Engine (TE)**：統一介面，跨 TCP / RDMA / CXL/shared-memory / NVMe-oF 做批次資料傳輸
- **P2P Store**：peer-to-peer object 共享（checkpoint 傳遞）
- **Mooncake Store**：分散式 KVCache 儲存引擎，多副本 + striped 並行 I/O
- **Mooncake-EP**：MoE 模型彈性 expert parallelism
- **KVCache-centric scheduler + prediction-based early rejection**：排程圍繞 cache 局部性而非單純 GPU 占用；超載時用預測模型早期拒絕，宣稱模擬 +525%、實際 +75% 吞吐
- 整合：vLLM、SGLang（HiCache 後端）、LMCache、LMDeploy、NIXL、TensorRT-LLM、vLLM-Ascend、xLLM
- 硬體：NVIDIA CUDA、AMD ROCm、華為 Ascend、Cambricon MLU、Moore Threads MUSA、MetaX MACA、T-Head PPU
- 傳輸：IB/RoCE RDMA、AWS EFA、NVLink、NVMe-oF、Barex、CXL

近 5 commit 觀察（2026-05-08~05-09，皆在 24h 內）：
```
2026-05-09 [TE] feat(transport): add independent maca_transport for Metax MACA C500 (#2059)
2026-05-09 [Store] Fix disk replica read paths for GPU KV cache (LOCAL_DISK zero-copy, DISK temp-buf …)
2026-05-08 [Store] fix(rust): add missing empty check for batch_is_exist (#2045)
2026-05-08 [engram] support engram (#1483)
2026-05-08 [Store] Fix SSD offload in Metadata Server mode (#193…)
```
活躍訊號真實，非殭屍 repo。

## 2. 對共腦的 leverage（4 個切點）

### 2-A. memory 系統收為 reference（**主切點**）

Mooncake 的 KVCache 池與共腦的 hot/warm/cold memory 是**結構同構**：

| Mooncake | 共腦 |
|---|---|
| GPU HBM（最熱、容量最小、頻寬最高） | CLAUDE.md + `memory/MEMORY.md` 索引（system prompt 自動載入，~200 行截斷） |
| CPU DRAM（次熱、容量大、頻寬中） | `memory/*.md` 個別檔（按需 Read） |
| SSD（冷、容量極大、頻寬低） | `projects/<name>/CONTEXT.md` + git history + archive/ |
| Transfer Engine（統一介面跨四層） | `lesson-scan.sh` + `Read` + `git log`（共腦的「跨層搬運器」） |
| KVCache-centric scheduler | 共腦 worker 派手前的「相關 lesson 掃描 + 焦點專案 + 容量檢查」 |
| Prediction-based early rejection | 共腦現缺失 — 沒有任何容量超載前的拒絕機制（候選改進） |

**借鏡點**（不照抄系統，借三個設計原則）：
1. **Cache 局部性是排程一等公民**：派 worker 時不只看「誰閒」，要看「相關 context 已載入哪個 substrate」
2. **多副本與容量上限是顯式設計**：MEMORY.md 200 行截斷該是 watermark，現在是硬切；缺 eviction 策略
3. **Prediction-based early rejection**：context 用量超過閾值前主動拒絕新 worker，避免硬截斷

### 2-B. spawn worker 編排

Mooncake prefill/decode 分離 → 共腦「重的派手到背景」已是 prefill/decode 雛形（Owner = decode，背景 worker = prefill）。Leverage：可借**單向 KV 流**概念，worker 完成後寫 `.worker-result.json` 是 prefill→decode handoff 的同構，schema 可加「context fingerprint」便於 decode 端跳過已熟脈絡。優先級 P2，現有機制夠用。

### 2-C. xray-skills.md 加關鍵字

加詞：`KVCache disaggregation` / `prefill-decode separation` / `prediction-based rejection` / `tiered memory pool` / `Transfer Engine pattern`。觸發場景：未來看到 inference infra 類 repo 才掃。優先級 P3。

### 2-D. skill metabolism

不適用。Mooncake 不是 skill 也不是 agent framework，與 5-substrate metabolism 不同維度。

## 3. 對 Owner 各 project 的 leverage 矩陣

| Project | Leverage | 動作 |
|---|---|---|
| **taichi-framework** | **HIGH** — KVCache 分層 = 共腦 memory tiering 的工業先例，FAST'25 best paper 有理論背書。可寫入 LANDSCAPE.md「memory architecture / tiered context」象限 | P0：抽 paper §3-4 的 tier 判據與 eviction 策略寫入 `docs/MEMORY-TIERING.md` |
| **local-distillation** | **MED** — Mooncake 整套太重（cluster 級 RDMA + NVMe-oF），單機蒸餾用不到。但 prefix cache 共享思想可用：local 模型若多 worker 同前綴（共腦標準 prompt 開頭重複），prefix-cache 可省 60%+ tokens | P2：local 模型跑通後評估 vLLM 內建 prefix caching 是否值得開 |
| **passive-income** | **LOW-MED** — Phase 3 SaaS 化才會碰；單機推論成本下降靠 model quantization 不靠 KV pool。若未來走 cloud serving，Mooncake 是學習對象不是依賴 | P3：歸檔備查，Phase 3 啟動時再讀 |
| **thesis** | **LOW** — 與 SSL pretrain / cucumber detection 無關 | 不採用 |
| **usv** | NONE | — |
| **drink-station / aquafeed / 其他** | NONE | — |
| **reading（第一性原理讀書會）** | NONE | — |

## 4. 全面風險評估

### 4-A. 技術風險：HIGH（若採用為基礎設施）

- C++ + RDMA + NVMe-oF + 多硬體後端，本質是**集群級系統**，完全超 Owner 一人組織硬體（RPi5 + 一台筆電）
- 編譯依賴與運維複雜度遠超 ROI
- **控制**：只借概念不借代碼/二進位

### 4-B. 商業授權：LOW

- Apache-2.0，可商用、可修改、可閉源衍生（保留版權聲明）
- 對 Owner passive-income SaaS 化或論文引用無法律阻礙

### 4-C. 路徑依賴：MED（若整合）

- 與 vLLM / SGLang / NIXL 深度耦合，採用後等於綁這條 stack
- **控制**：不採用為基礎設施；僅作為文獻參考與架構學習

### 4-D. 政治供應鏈：MED-HIGH（軟體本身 LOW，但需聲明）

- Moonshot AI = 北京月之暗面，中國公司
- **代碼本身**：Apache-2.0 + 學術論文（FAST'25 Best Paper）+ 多家硬體商整合，技術可信度高
- **二進位 / 預編譯**：若採用任何 prebuilt artifact 需評估供應鏈（CloakBrowser 教訓的同類風險）
- **API 整合**：若整合 Kimi API 觸碰中國服務 → 客戶端紅線，passive-income 對歐美客群須避開
- **學術引用**：完全 OK，FAST'25 是 USENIX 旗艦，引用無立場問題
- **控制**：本研究取**僅讀概念與論文，不下載任何 Mooncake binary、不調用 Kimi API**

### 4-E. 社群與知識資源：LOW（資源豐富）

- 5.3k stars / 729 forks / 428 open issues / has_discussions / has_wiki
- FAST'25 Best Paper + arxiv 詳細技術報告
- 整合方（vLLM / SGLang / 多家硬體）多樣，知識散布廣
- 學習成本：高（需讀懂分散式系統 + GPU 推論），但有 paper 可繞過 code

## 5. 自洽校核

對照 memory 三條：

- **`project_shared_brain`（共腦架構）**：本研究強化此 memory — KVCache 分層提供工業實證，hot/warm/cold tier 不是憑空設計。**不修改 memory**，但在 LANDSCAPE.md 補對照即可。
- **`project_skill_metabolism`（5 substrate + birth filter + demotion）**：Mooncake 的 eviction 策略可借鏡 demotion 設計 — 目前共腦 demotion 是「使用頻率低」單一維度，Mooncake 是「最近使用 + cache 命中率 + 容量壓力」三維。**P2 改進候選**，但非緊急。
- **`feedback_session_restart_memory_model`（重啟前做落地檢查）**：Mooncake 啟示 — session restart 不是「丟記憶」而是「evict from HBM to DRAM/SSD」。Owner 怕重啟丟洞察 → 引導落地是把熱資料「flush 到下一層」的同構操作。**強化既有 feedback，不新增**。

無衝突。Mooncake 是強化型參考，不引發 memory 改寫。

## 6. 立即可動手 actions（低承諾、可逆、無付費 / 無新硬體）

| # | Action | 承諾 | 可逆 | 動作 |
|---|---|---|---|---|
| A1 | 讀 FAST'25 paper §3-4，抽出 tier 邊界判據與 eviction 策略，寫入 `projects/taichi-framework/docs/MEMORY-TIERING.md`（草稿即可） | ~1 hr 閱讀 + 30 min 寫作 | Y | 下次 Owner session 自決 |
| A2 | LANDSCAPE.md 補「memory architecture / tiered context」象限對照段，列 Mooncake | 15 min | Y | 下次 Owner session 自決 |
| A3 | xray-skills.md 加關鍵字 `KVCache disaggregation` / `tiered memory pool` / `prefill-decode separation` | 5 min | Y | 下次 Owner session 自決 |
| A4 | 讀 README + 跑 `gh api repos/kvcache-ai/Mooncake/issues` 看 community pain points，找出 production 落地時最常踩的坑（純情報收集，不部署） | 30 min | Y | 視 A1 之後是否有興趣再做 |
| A5 | 對共腦現行 `MEMORY.md ≤200 行截斷` 補 watermark / eviction 設計提案（受 Mooncake `prediction-based early rejection` 啟發） | 30 min | Y | A1 後若決定深化再做 |

**禁忌動作**（明示）：
- 不下載任何 Mooncake prebuilt binary
- 不在 RPi5 / 筆電編譯 Mooncake（資源遠不足，無意義）
- 不調用 Kimi API（中國服務，passive-income 客群紅線）

## 7. 結論（But-For）

**But-For**：若沒有讀 Mooncake，共腦 memory tiering（hot/warm/cold）會繼續停留在「直覺設計+CLAUDE.md 經驗法則」階段；讀了之後得到 FAST'25 Best Paper 級別的工業證據與術語體系（KVCache pool / prefill-decode disaggregation / cache-centric scheduling / prediction-based early rejection），可以把共腦 memory 從「規則式分層」升級為「容量+局部性+預測拒絕」三維治理。Leverage **概念層 HIGH，代碼層 ZERO**——不採用、不整合、不依賴任何 Moonshot 出產的二進位或 API；僅取論文與 README 概念，作為共腦 memory 工程的文獻錨點。本研究產出三件可動作（A1/A2/A3 即可在 30 分鐘 Owner session 內完成），其餘（A4/A5）視 Owner 興趣推進。

Mooncake 的核心啟示一句話：**「排程不是看誰閒，是看相關 context 已經在哪一層 cache。」** 這對共腦 worker 派手機制是直接同構。
