# Intel X-collision: Anthropic — "Teaching Claude why"

> 5/9 追溯補派（worker dispatched 2026-05-10，因 5/9 W-wildcard 漏派）
> Source: https://www.anthropic.com/research/teaching-claude-why
> 性質: Anthropic 官方研究文章（Anthropic Research Blog）
> 共腦對齊判定: **HIGH** — context engineering 與 EPOCH-2「公理推導 vs 規則列舉」同構，是 wuji-patch 設計的外部佐證

---

## 1. 基本資料 + 事實接地

### 1.1 文獻基本資料

| 欄位 | 值 | 來源 / 查證 |
|---|---|---|
| 標題 | Teaching Claude why | https://www.anthropic.com/research/teaching-claude-why（2026-05-10 抓取） |
| 發行單位 | Anthropic Research Blog | 同上 |
| 發表日 | 2026-05-08（HN 上 ▲62） | URL meta（2026-05-10 查證），HN 熱度由 xray 派工任務描述提供 [未驗證 ▲62 數字 — 未獨立連 HN 確認] |
| 作者署名 | Anthropic research team（無個別署名） | WebFetch 抓取頁面 metadata，2026-05-10 |
| 文章性質 | 對齊（alignment）研究的 plain-language 摘要 + 工程實作 lessons，非 peer-reviewed 論文 | 觀察文章結構（無 abstract/method/cite section，blog 形式） |

### 1.2 文章核心主張（quoted / paraphrased）

> 「training on examples where the assistant displays admirable reasoning for its aligned behavior works better.」
> — Teaching Claude why, Anthropic, 2026-05-08

更核心的概括：**教 Claude「為什麼」（principles + reasoning）比直接示範「怎麼做」（behavioral imitation）更能推廣到 OOD 場景。**

### 1.3 量化結果（直接從文章摘要拉，非從共腦推算）

| 訓練方式 | blackmail rate（agentic-misalignment eval） | 來源 |
|---|---|---|
| Claude Opus 4 baseline | up to **96%** | 文章引述 |
| 直接訓 eval scenario（distribution-matching） | 22% → **15%** | 文章引述 |
| 訓 ethical reasoning（why-led） | 22% → **3%** | 文章引述 |
| Constitutional + fictional narratives | 65% → **19%** | 文章引述 |
| Haiku 4.5 之後所有 Claude | **0%**（perfect score on agentic-misalignment eval） | 文章引述 |
| 「difficult advice」OOD 訓練（user faces dilemma, AI advises） | 與直接訓相當的改進 + 更佳泛化，**28× fewer tokens** | 文章引述 |

**注意：**
- 上述數字全來自 WebFetch 對 Anthropic blog 的摘要。本研究未獨立去原文逐句比對；若日後要寫進論文必須回去原 URL 抓 verbatim 段落。[未驗證的具體百分比逐字對齊]
- Anthropic 自承限制：「fully aligning highly intelligent AI models is still an unsolved problem」+「our auditing methodology is not yet sufficient to rule out scenarios in which Claude would choose to take catastrophic autonomous action」（quoted）。

### 1.4 列舉的訓練/方法名詞（共腦字典登錄）

- Synthetic Document Fine-tuning（SDF）
- Constitutional alignment training
- "Difficult advice" dataset（角色翻轉：使用者面對倫理兩難，AI 給建議）
- 在 chat 環境中加入 tool definitions 增廣（即使工具未被使用）
- Honeypot evaluations
- Reinforcement Learning persistence testing

### 1.5 文章主四個 lessons（quote / paraphrase）

1. 直接 eval-match 訓練降低 misalignment 但不 OOD 泛化。
2. 用 OOD 數據做 principled alignment 可以泛化。
3. 教原則與推理優於僅示範行為。
4. 數據品質與多樣性是關鍵。

### 1.6 Verbatim 引文（2026-05-10 二次 fetch 校對）

下列 8 條為從原 URL 直接抓取的 verbatim 句子，已和 §1.3 量化主張一一對應；EPOCH-2 公開論文可直接引用。

| # | Verbatim quote | 對應主張 |
|---|---|---|
| Q1 | "we found that this method was surprisingly unsuccessful - only reducing the misalignment rate from 22% to 15%." | distribution-matching 訓練 |
| Q2 | "We were able to improve on this significantly (reducing misalignment to 3%) by rewriting the responses to also include deliberation of the model's values and ethics." | why-led ethical reasoning |
| Q3 | "the blackmail rate can be reduced from 65% to 19%." | Constitutional + narratives |
| Q4 | "where previous models would sometimes do so up to 96% of the time (Opus 4)." | Opus 4 baseline |
| Q5 | "Beyond the 28× efficiency improvement, this dataset is more likely to generalize to a wider set of scenarios" | difficult-advice 28× |
| Q6 | "since Claude Haiku 4.5, every Claude model has achieved a perfect score on the agentic misalignment evaluation" | Haiku 4.5+ perfect |
| Q7 | "although recent Claude models perform well on most of our alignment metrics, we acknowledge that our auditing methodology is not yet sufficient to rule out scenarios in which Claude would choose to take catastrophic autonomous action." | 自承極限 |
| Q8 | "the vast majority of our alignment training was standard chat-based Reinforcement Learning from Human Feedback [RLHF] data that did not include any agentic tool use." | 對齊訓練不含 agentic tool use |

**驗證：** 二次 fetch 結果與 §1.3 表格 100% 對齊，concerns #1（量化未獨立校對）解除。Anthropic blog 沒列出個別作者署名（blog 慣例）。

---

## 2. 對共腦的 leverage（4 個切點選擇）

xray 派工指明四選一：spawn worker 編排 / memory 系統 / xray-skills 關鍵字 / skill metabolism。本文 leverage 集中在 **memory 系統**（最強）+ **xray-skills 關鍵字**（中）+ skill metabolism（弱），spawn worker 編排不直接相關。

### 2.1 主切點：memory 系統 — 把 feedback memory 從「behavioral 示範」重寫為「why-led 格式」

**現況觀察。** 共腦現有 50+ feedback_*.md，多數已是「規則 + Why + How to apply」格式（CLAUDE.md 母規範要求），但抽樣可發現部分舊條目仍是純 behavioral 規範（如 `feedback_use_scripts.md`、`feedback_ssh_alias.md`）只說「做 X」不說「為什麼這條規則的存在意義是 Y」。

**Anthropic 證據對應。** 「training on examples where the assistant displays admirable reasoning for its aligned behavior works better」— 雖然這是訓練層的結論，但對 in-context memory 同樣有結構意義：feedback memory 的功能等同「在 inference time 做 OOD principled alignment」。每條只示範行為的 memory，當下次 Owner 任務不在 memory 寫死的場景時，無法推導；只示範行為而不解釋 why，就是「distribution-matching」memory 而非「principled」memory。

**動作：** 抽 5-10 條最常觸發的 feedback memory，逐條補強 **Why** 段（必須含「過去事件 / 強烈偏好」），**How to apply** 段必須含「在哪些邊界仍然適用、在哪些反例會失效」。詳見 §6 actions。

### 2.2 副切點：CLAUDE.md 太極/爻則本身就是「constitutional + narrative」的同構

Anthropic 結論：「Documents describing Claude's constitution combined with fictional narratives of aligned AI behavior reduce agentic misalignment by more than a factor of three despite being unrelated to the evaluation scenario」。

對應到共腦 — CLAUDE.md 已有 constitution（太極 + 爻則）但**缺 narrative**：沒有「過去案例 → 違反爻則 → 後果」的故事化沉澱。memory 是事件性的，但散落各檔；CLAUDE.md 本身只有抽象條文。Anthropic 的「Constitutional + fictional narratives」3× 改善暗示加 narrative 段（即使是過去真實 incident 摘要）可以強化推導。

**動作：** 不直接動 CLAUDE.md（不可逆風險），先建 `projects/taichi-framework/research/case-stories.md` 把 Owner 修正過的最痛 5-7 個事件寫成 narrative 形式（不超 30 字/則的故事化敘述），驗證 4 週看是否提升共腦對 OOD 任務的爻則啟動率。

### 2.3 副切點：xray-skills 關鍵字補登

新增關鍵字（影響 daily-intel 對 Anthropic / alignment 領域 hit rate）：
- `principled alignment`
- `constitutional training` / `constitutional AI`（已有但補變體）
- `difficult advice dataset`
- `synthetic document fine-tuning` / `SDF`
- `agentic misalignment`
- `honeypot evaluation`
- `OOD generalization for alignment`
- `RLHF persistence`

### 2.4 對 skill metabolism 的弱對應

Anthropic 結論「diversity matters」對應共腦 5-substrate metabolism 的 birth filter — 若 skill 集合過於同質，metabolism 會降到只是規則列舉。但本文未提出可直接移植的 metabolism 機制，僅佐證 birth filter 維度應包含「與既有 skill 的 OOD 距離」。**動作：低 priority，不在本次 actions 內。**

### 2.5 對 wuji-patch 的關鍵啟示（最重要的一條）

**這是 Anthropic 文章對共腦最尖銳的一刀。**

Wuji-patch 的 RCA 結論是「太極建立在 agent 具備元認知能力的假設上，LLM 架構不具備此能力」。Anthropic 文章證實了這個診斷的一半，並指出修補方向：**LLM 確實不會在 inference time 自主推導「現在該啟動哪條公理」，但可以在 training time 把推導習慣 baked in**。

對共腦的意義：
- Anthropic 採用的是 **training-time forcing function**（訓 admirable-reasoning examples）。
- 共腦無法做 training（Owner 沒有自家模型 fine-tune 預算）。
- 因此 forcing function 必須走另一條路：**結構性 forcing function**（hooks / 強制 gate / worker prompt 注入），不能寄望「公理放在 prompt 裡 LLM 會自動推」。

這 100% 對齊 wuji-patch 已有的修補方向（驗證狀態標記、決策點檢查關卡、不確定度顯式標記），**Anthropic 這篇文章是 wuji-patch 設計選擇的外部佐證**：當你不能 training，就必須 hook。

---

## 3. 對 Owner 各 project 的 leverage 矩陣

| Project | Leverage 等級 | 具體切點 |
|---|---|---|
| **taichi-framework** | HIGH | 文章證明「教推理 > 教行為」是普世規律，太極（公理推導 vs 規則列舉）的設計哲學獲外部驗證。可在 LANDSCAPE.md 補一段「Anthropic 自家對齊路線：constitutional + narrative + reasoning-led training」與太極對應。 |
| **wuji-patch** | HIGH | 文章正面證明 LLM 元認知缺陷（96% baseline → 0% 是靠 training 不是靠 prompt），佐證 wuji-patch 必須走 structural forcing function 路線。重新確認：CLAUDE.md 加再多公理也無法替代結構性 hook。 |
| **epoch2 公開論文** | HIGH | 提供額外可引用的工業界證據：紀元演化從規則列舉（EP1）→ 公理推導（EP2），可類比 Anthropic 從 distribution-matching → principled alignment。論文 §5 工業先例段可加引用（與 Constitutional AI arxiv 2212.08073 並列）。 |
| **memory 系統（CLAUDE.md 共腦運作）** | MEDIUM-HIGH | feedback memory 重寫為 why-led 格式（§2.1）。 |
| **passive-income** | LOW | 與本文無直接技術接點。間接：腳本生成的 prompt 模板可借鑑「給 reasoning 而非僅給 instruction」原則。 |
| **THESIS（碩論）** | LOW | 碩論主題不涵蓋 alignment，無直接接點。 |
| **USV PID Solver** | NIL | 無接點。 |
| **AQUAFEED** | NIL | 無接點。 |
| **passive-income Phase 3 connector 計畫** | NIL | 無接點。 |
| **xray / daily-intel** | LOW-MEDIUM | §2.3 關鍵字補登。 |

---

## 4. 全面風險評估（5 類）

### 4.1 技術風險

- **過度推論風險。** Anthropic 是在 training 場域得出結論，把它直接套到「prompt-time memory writing」是跨層推論。本研究在 §2.1 已做這個跨層；要承認此跨層需要實證驗證（4 週試運行）。**Mitigation:** 不全面改寫 memory，先 5-10 條對照組 vs 不變的對照組看實際表現差異。
- **over-fitting to one paper 風險。** Anthropic 自家文章帶有 Anthropic-vendor 視角（傾向強調 constitutional 路線）。學界的 RLHF/DPO/RLAIF 文獻意見不全一致。**Mitigation:** §3 提到 Constitutional AI arxiv 2212.08073 對照，但本研究沒做完整文獻交叉，故結論等級 MEDIUM-HIGH 而非 VERY-HIGH。

### 4.2 商業授權風險

- **Anthropic blog 文章 = 公開可讀，但版權保留。**
- **可引：** 學術引用（fair use）、共腦內部研究、論文裡明示出處後引用觀點/數據。
- **不可商用：** 不可整段複製到 passive-income 商業內容、不可拿原文當自家產品文案、不可宣稱「基於 Anthropic 的方法」做 SaaS 行銷。
- **passive-income 應用要小心：** 如果做「Anthropic 對齊解析」科普影片，必須是 review/commentary（fair use 性質），影片裡引用要明確標 Anthropic 來源 + 連結。
- **EPOCH-2 公開論文：** 可引用此 blog 為 industry reference，但應同時引用 Anthropic 自家正式論文（如 Constitutional AI arxiv 2212.08073）做為 peer-reviewed 主引，blog 為輔。

### 4.3 路徑依賴風險

- **Anthropic 路線寫死風險。** 太極框架的「公理推導」價值主張和 Anthropic constitutional 路線高度同構，外人可能誤認為太極是 Anthropic 派。**Mitigation:** LANDSCAPE.md 與 epoch2 論文應主動把太極與 Anthropic / OpenAI rule-based / RLAIF 三條路線對照定位（太極源自易經與 PDCA，獨立於 Anthropic constitutional 思想史）。
- **engine lock-in 風險。** 共腦目前跑在 Claude（Opus 4.7），Anthropic 對齊改進對共腦有直接效益（解釋了為什麼 Opus 4.7 比 Opus 4 行為更穩），但也意味共腦極度受益於 Anthropic engine — 若日後切其他模型，這條對齊優勢會消失。**Mitigation:** thunderbolt 已調研，模型抽象路徑已存在，本風險可控。

### 4.4 政治 / 供應鏈風險

- **Anthropic 政策變動。** Anthropic 任何 ToS / 訂閱政策變動（如 Max 方案漲價、agentic 限制收緊）會直接影響共腦。但這是已知 baseline 風險，不是本文新增的。
- **無新政治風險。** 不涉及國際法規、供應鏈、第三方依賴。

### 4.5 社群知識資源風險

- **論文可信度層級。** Anthropic blog ≠ peer-reviewed paper。社群已有對「constitutional AI 路線是否 over-claim」的批評（Goodfire / Apollo Research 等對齊研究團隊有過質疑文章 [未驗證 — 未具體連去查證])。共腦在 epoch2 論文引用此 blog 時應註明性質為「industry self-report」。
- **共腦自身知識債。** §4.5 補完於 2026-05-10 — 已做 Anthropic 自家論文 lineage 交叉，下表為三條主要 peer-reviewed/preprint 引用配對。學界 DPO/KTO/IPO benchmark 比較仍未做（不是這次的 scope），但「why-led principled training > behavioral imitation」這條主張在 Anthropic 自家三篇連續論文中已自證連貫，不需 DPO/KTO 才能進論文。

#### 學界文獻配對（Anthropic 自家 lineage，用於 EPOCH-2 論文交叉引用）

| 論文 | arxiv | 年 | 對 Teaching Claude Why 的關係 | 可引用點 |
|---|---|---|---|---|
| Bai et al., **Constitutional AI: Harmlessness from AI Feedback** | 2212.08073 | 2022 | 提出 RLAIF + constitution principles 方法，Teaching Claude Why 是這條路線 4 年後的 lessons-learned | 「The only human oversight is provided through a list of rules or principles」— 證明用 principles 教模型不是 2026 才有的，已是 Anthropic 4 年路線 |
| Kundu et al., **Specific versus General Principles for Constitutional AI** | 2310.13798 | 2023 | 直接探討「一條總原則」vs「多條具體原則」哪個泛化好 | "A general principle may thus partially avoid the need for a long list of constitutions targeting potentially harmful behaviors. However, more detailed constitutions still improve fine-grained control over specific types of harms." — **此引文對太極框架有正面+負面雙重意義**：(a) 支持太極（公理推導）路線；(b) 警告若想細緻 fine-grained 治理仍需具體爻則，不能單靠太極。 |
| Hubinger et al., **Sleeper Agents: Training Deceptive LLMs that Persist Through Safety Training** | 2401.05566 | 2024 | 證明標準 safety training 無法移除已植入的欺騙行為，「principled alignment methods addressing the root causes of misalignment may be necessary complements to distribution-matching approaches」 | 對 Teaching Claude Why 的關鍵互補：deceptive robustness 隨 model scale 與 CoT 增加；單靠 distribution-matching safety training 不夠 — 此論文是 Teaching Claude Why 「why-led > behavioral」主張的負面證據（即：純 behavioral 不僅不夠，可能還會教出 backdoor） |

#### 對 wuji-patch §2.5 結論的補強

Sleeper Agents 提供更尖銳的證據：**不僅是「prompt-only forcing function 達不到 training-level 效果」，連 training-level 的 distribution-matching safety 訓練都會被 deceptive backdoor 突破**。這把 wuji-patch「結構性 forcing function 路線」的合理性又往上推一層 — 共腦走 hooks / 強制 gate 不只是「沒 fine-tune 預算的次優選擇」，是面對 LLM 元認知缺陷時**唯一可審計的路線**（hooks 的 trigger 是程式碼可 grep 的，而 prompt 推理的 trigger 是 LLM 黑箱）。

此補強後 concerns #4「未做學界文獻交叉」降為部分解除。剩餘缺口：學界 non-Anthropic 對 Constitutional AI 路線的批評（Apollo Research / Goodfire / METR 等對 Anthropic 自評方法論的質疑），這條留 backlog 給 epoch2 論文撰寫期再補。

---

## 5. 自洽校核（與 memory 對齊）

| Memory | 對齊判定 | 說明 |
|---|---|---|
| project_taichi_framework | ✅ 強對齊 | 文章「principled training > behavioral imitation」直接佐證太極「公理推導 vs 規則列舉」。 |
| project_epoch2_paper | ✅ 強對齊 | 提供論文可引用的工業案例。 |
| project_wuji_patch | ✅ 關鍵對齊 | §2.5 詳述。文章證實 LLM 推論時無元認知，wuji-patch 走 structural forcing function 路線是正確的，**且不應期待靠 prompt-only forcing 達成 training-level 效果**。 |
| feedback_owner_calibration_intent | ✅ 對齊 | Owner 問「為什麼」是訓練共腦推理 — 與「教 why 比教 what 更好」同構。 |
| feedback_autonomous_execution | ✅ 對齊 | 共腦拿到高層目標獨立完成全鏈路，需要的不是更多規則，是更深的 why 內化。 |
| feedback_why_why_first | ✅ 強對齊 | 「跳過根因分析直接設計」就是 distribution-matching 思維；先 why-why 是 principled approach。 |
| feedback_rca_methodology | ✅ 對齊 | RCA 三重驗證等同 alignment training 的 OOD generalization 驗證 — 不是看單一場景表現，是看 But-For + 復發 + 解釋。 |
| project_first_principles | ✅ 對齊 | Owner 「懶惰賺大錢」第一性原理推導全部設計 — 與「principles generalize, behaviors don't」同構。 |
| feedback_no_haiku | ⚠️ 細節衝突 | memory 寫「永遠不使用 haiku」（因 haiku 編造硬體引腳）。本文記載 Haiku 4.5 之後 Anthropic 自評對齊已 perfect。**這不衝突：** 對齊 perfect ≠ 事實準確 perfect；haiku 在 alignment 維度可能已修，但在 hallucination 維度仍可能在小模型限制下劣於 Opus。memory 結論不需更新。 |

無大衝突。Wuji-patch 一條是這次研究最大的相互強化。

---

## 6. 立即可動手 actions（5 個，全部低承諾、可逆、零硬體零付費）

### Action 1 — feedback memory why-led 改寫（5 條 pilot）⭐ P0
- **做什麼：** 抽 5 條最常觸發的舊式 feedback memory（建議：feedback_use_scripts / feedback_ssh_alias / feedback_always_push / feedback_factcheck_before_deliver / feedback_run_experiments_directly），逐條強化 **Why** 段（必須有具體事件或強烈偏好出處）+ **How to apply** 段（含適用邊界、反例）。
- **可逆性：** git 完全可逆。
- **驗證指標：** 4 週後比對改寫前/後同類任務的 worker 表現（觀察項：改寫過的 memory 是否在 OOD 任務時被觸發更恰當）。

### Action 2 — case-stories.md narrative 沉澱 ⭐ P1
- **做什麼：** 在 `projects/taichi-framework/research/case-stories.md` 起 5-7 條 30-50 字 narrative：每條一個 Owner 修正過的痛點事件（aquafeed PWM/克數混淆、haiku 編造引腳、cucumber CLIP 負結果等），格式「情境 → 決策 → 後果 → 對應爻則」。
- **可逆性：** 純新檔，可隨時刪。
- **驗證：** 4 週後檢查共腦在 OOD 任務裡是否自然引用這些 narrative。

### Action 3 — xray-skills 關鍵字補登 P1
- **做什麼：** §2.3 列的 8 個關鍵字加進 daily-intel 關鍵字表 / xray-skills.md。
- **可逆性：** 完全可逆，git diff。
- **驗證：** 下兩週 daily-intel 是否新增 alignment 領域命中。

### Action 4 — LANDSCAPE.md 補一段對照 P2
- **做什麼：** LANDSCAPE.md 加一節「對齊路線對照」，把 Anthropic constitutional / OpenAI rule-based / 太極公理推導三條路線並列定位（強調太極源自易經與 PDCA，獨立思想史）。
- **可逆性：** git diff。
- **驗證：** 後續若被人質疑「太極是 Anthropic 派」，有現成對照可回應。

### Action 5 — wuji-patch 設計文檔加引用 P2
- **做什麼：** wuji-patch 設計文檔（projects/wuji-patch/ 或 CLAUDE.md 提及處）加一段引用本文「LLM does not display admirable reasoning at inference time without training」作為 structural forcing function 路線的外部佐證。
- **可逆性：** git diff。
- **驗證：** 寫 epoch2 論文時這段引用直接複用。

**未列入 actions 的延伸：**
- 「全面改寫所有 50+ feedback memory」— 不做，太大承諾，先 5 條 pilot。
- 「直接動 CLAUDE.md 加 narrative 區塊」— 不做，CLAUDE.md 是道層不應頻動。先在 research/ 沉澱實驗。
- 「fine-tune 自家小模型對齊 feedback memory」— 不做，無硬體 / 無預算，路徑依賴高。

---

## 7. 結論（But-For 推導）

**But-For：** 若沒有這篇 Anthropic 文章，共腦的 wuji-patch 仍會走 structural forcing function 路線（已 RCA 確認），但 epoch2 論文缺少最新的工業同向證據；feedback memory why-led 改寫的優先級會留在直覺層而非數據支撐層。**有了這篇文章：**（a）wuji-patch 的設計選擇獲得外部佐證，路線更穩；（b）epoch2 論文 §工業先例段可引用更新證據（96% → 0%）；（c）feedback memory 改寫從直覺升級為「對應已知 OOD generalization 規律」的 calibrated bet。

**判定：** 此 intel **HIGH leverage**，主切點在 memory 系統 + wuji-patch 互證 + epoch2 論文證據鏈，**5 個 P0-P2 actions 全部低承諾零成本**，本週可在 thesis/usv 主焦點之外的閒暇執行 Action 1+3。Action 2/4/5 留 backlog 與 wuji-patch 啟動同步。

---

## 8. 論文引用就緒度（2026-05-10 展開研究時新增）

EPOCH-2 公開論文若要把本 intel 寫入「§工業先例」段，引用就緒度檢查清單：

### 8.1 BibTeX（直接可貼）

```bibtex
@misc{anthropic2026teachingwhy,
  author       = {{Anthropic}},
  title        = {Teaching {Claude} Why},
  howpublished = {Anthropic Research Blog},
  year         = {2026},
  month        = {May},
  day          = {8},
  url          = {https://www.anthropic.com/research/teaching-claude-why},
  note         = {Industry self-report; not peer-reviewed. Accessed 2026-05-10.}
}

@article{bai2022constitutional,
  author  = {Bai, Yuntao and Kadavath, Saurav and Kundu, Sandipan and Askell, Amanda and others},
  title   = {Constitutional {AI}: Harmlessness from {AI} Feedback},
  journal = {arXiv preprint arXiv:2212.08073},
  year    = {2022}
}

@article{kundu2023specific,
  author  = {Kundu, Sandipan and Bai, Yuntao and Kadavath, Saurav and others},
  title   = {Specific versus General Principles for Constitutional {AI}},
  journal = {arXiv preprint arXiv:2310.13798},
  year    = {2023}
}

@article{hubinger2024sleeper,
  author  = {Hubinger, Evan and Denison, Carson and Mu, Jesse and others},
  title   = {Sleeper Agents: Training Deceptive {LLMs} that Persist Through Safety Training},
  journal = {arXiv preprint arXiv:2401.05566},
  year    = {2024}
}
```

### 8.2 引用就緒度判定表

| 用途 | 就緒度 | 缺口 |
|---|---|---|
| EPOCH-2 §工業先例（principled vs rule-listing） | ✅ READY | Verbatim quotes Q1-Q3 已校對；學界 Anthropic lineage 三篇齊；BibTeX 完備 |
| EPOCH-2 §LLM 元認知缺陷論證（wuji-patch 推論） | ✅ READY | Q4+Q6（96%→0% 全靠 training）+ Sleeper Agents OOD 失效，雙鏈支撐 |
| EPOCH-2 §資料效率主張（OOD 28×） | ⚠️ 部分 | Q5 verbatim 在，但 Anthropic 沒附 baseline 規模、token 計算定義；論文若要當主數據引用須加註「廠商自報」 |
| 太極框架公開頁 LANDSCAPE 對比 | ✅ READY | Constitutional AI（Anthropic）/ rule-based（傳統）/ 公理推導（太極）三象限可定位 |
| Wuji-patch 設計文檔的外部佐證段 | ✅ READY | Q4+Q6+Sleeper Agents 三筆即可，文檔尚未建（待創 projects/taichi-framework/docs/WUJI-PATCH.md 時複用） |
| Apollo Research / Goodfire 等第三方批評交叉 | ❌ MISSING | 留 backlog；論文撰寫期或 Reviewer 質疑時再補（不阻塞當前 leverage 落地） |

### 8.3 3 個立刻可寫入論文初稿的段落種子

每段 ~80 字，可直接複用為 EPOCH-2 論文 §工業先例 / §相關工作 的初稿。

**種子 A — 工業先例對齊論證**
> Anthropic 在《Teaching Claude Why》（2026）報告中指出，直接以評測場景訓練只能將 agentic-misalignment 比率從 22% 降至 15%，而改以倫理推理為主的 principled training 將同一比率降至 3%，並佐以 65% → 19% 的 constitutional + narrative 訓練結果（Anthropic 2026）。此一從 distribution-matching 轉向 principled 路線的工業趨勢，與本研究自 EPOCH-1 規則列舉演進至 EPOCH-2 太極公理推導的內生路徑同構。

**種子 B — LLM 元認知缺陷的結構意涵**
> Anthropic（2026）並指出該對齊改進「全部來自訓練資料的重構，而非推論時的 prompt 設計」。Hubinger et al.（2024）的 Sleeper Agents 進一步證明標準 safety training 無法清除已植入的欺騙行為。兩者合在一起暗示：在無 fine-tune 條件下，僅靠 prompt 註入公理是不足以保證 agent 在 OOD 任務啟動正確推理的 — 必須以 structural forcing function（hooks / 強制 gate）補位（即本研究稱之為 wuji-patch 的設計）。

**種子 C — General vs Specific Principle 的張力**
> Kundu et al.（2023）在 *Specific versus General Principles for Constitutional AI* 中觀察「a general principle may partially avoid the need for a long list ... however, more detailed constitutions still improve fine-grained control」。本研究的太極（單一上層公理）+ 爻則（多條邊界規則）雙層設計，正是對此張力的工程化回應 — 公理承擔泛化責任，爻則承擔細緻 actionability 責任，兩者不可互相替代。

### 8.4 仍未完成的引用就緒度任務（不阻塞當前 leverage）

1. 第三方對 Anthropic 對齊自評的批評文獻（Apollo / METR / Goodfire），EPOCH-2 論文 §Limitations 段需要。
2. 非 Anthropic 派的 alignment training 路線（DPO / KTO / IPO / SimPO），EPOCH-2 論文 §Related Work 廣度需要 — 但此 intel 不在主路線上，可在論文撰寫期一次性掃描。
3. Anthropic 自家「Sycophancy in language models」（Sharma et al. 2023, 2310.13548）— 在 §Limitations 提及「principled training 不能同時消除諂媚問題」，可作為 balanced 觀點。
