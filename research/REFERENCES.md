# 學術引用

> 太極框架引用的理論來源與學術文獻。
> 最後更新：2026-05-10（補 AI 對齊類別 ×3：Constitutional AI / Sleeper Agents / Teaching Claude Why）

---

## 經典文獻

### 易經

- 《周易》繫辭上傳
  - 「易有太極，是生兩儀，兩儀生四象，四象生八卦。」
  - 框架的核心生成結構：一條公理→二元分化→四種模式→無限行為。

- 《周易》繫辭下傳
  - 「生生之謂易。」
  - 框架的動態循環（感→化→貞→新）的理論根據。

- 《周易》繫辭上傳
  - 「寂然不動，感而遂通天下之故。」
  - 框架的觸發式運作模型：零能耗待機，接觸刺激後從公理即時推導。

---

## 管理學與知識理論

### Deming, W. Edwards

- Deming, W.E. (1986). *Out of the Crisis*. MIT Press.
- PDCA 循環（Plan-Do-Check-Act）與生生循環的同構映射：感=Plan，化=Do，貞=Check，新=Act。

### Hayek, Friedrich A.

- Hayek, F.A. (1945). "The Use of Knowledge in Society." *American Economic Review*, 35(4), 519-530.
- https://www.jstor.org/stable/1809376
- 分散知識無法被中央完整收集的論證。規則式治理（中央列舉）必然失敗的理論基礎。太極如同價格訊號——用最壓縮的資訊協調全局行為。

### Nonaka, Ikujiro

- Nonaka, I. (1995). *The Knowledge-Creating Company: How Japanese Companies Create the Dynamics of Innovation*. Oxford University Press.
- SECI 模型（社會化→外化→組合→內化）與框架的知識轉化對應：冷啟動校準=外化（隱性→顯性），使用中校準=內化（顯性→隱性），爻則壓縮=組合（顯性→顯性），日常推導=社會化（隱性→隱性）。

---

## 物理學

### Wilson, Kenneth G.

- Wilson, K.G. (1971). "Renormalization Group and Critical Phenomena. I. Renormalization Group and the Kadanoff Scaling Picture." *Physical Review B*, 4(9), 3174-3183.
- DOI: 10.1103/PhysRevB.4.3174
- 重整化群理論：微觀的大量參數在宏觀尺度坍縮為少數有效算符。規則→爻則→太極的壓縮過程是同一件事——去掉不影響宏觀行為的細節，保留決定系統行為的少數關鍵參數。

---

## 心理學與價值理論

### Schwartz, Shalom H.

- Schwartz, S.H. (1992). "Universals in the Content and Structure of Values: Theoretical Advances and Empirical Tests in 20 Countries." *Advances in Experimental Social Psychology*, 25, 1-65.
- DOI: 10.1016/S0065-2601(08)60281-6
- 基本價值理論：10 種跨文化普遍價值的圓形結構。框架的校準題庫設計參考了 Schwartz 的維度覆蓋，但不使用固定分類——維度從使用者回答中湧現。

### Enneagram

- Riso, D.R. & Hudson, R. (1999). *The Wisdom of the Enneagram*. Bantam Books.
- 九型人格的核心慾望/核心恐懼結構。框架的 Layer 1 校準（找兩儀）的題目設計參考了核心慾望（Q1）和核心恐懼（Q2）的提問方式。

### Big Five / OCEAN

- John, O.P. & Srivastava, S. (1999). "The Big Five Trait Taxonomy." *Handbook of Personality: Theory and Research*, 2, 102-138.
- 五大人格特質模型。框架的校準題庫設計確保五個維度（開放性、盡責性、外向性、親和性、神經質）都被至少一題涵蓋，避免系統性盲點。

---

## 強化學習

### Sutton, Richard S. & Barto, Andrew G.

- Sutton, R.S. & Barto, A.G. (2018). *Reinforcement Learning: An Introduction* (2nd ed.). MIT Press.
- http://incompleteideas.net/book/the-book-2nd.html
- 框架的校準機制本質上是強化學習：太極+爻則+記憶=策略（Policy），使用者糾正=負獎勵，使用者確認=正獎勵，回饋壓縮=策略更新。差異在於公理作為強壓縮的先驗知識，讓系統從少量回饋就能大幅調整行為。

---

## AI 模擬與對齊

### Park, Joon Sung et al.

- Park, J.S. et al. (2023). "Generative Agents: Interactive Simulacra of Human Behavior." *UIST 2023*.
- DOI: 10.1145/3586183.3606763
- 生成式 agent 模擬人類行為的開創性工作。後續 2024 年 DeepMind 擴展研究透過 2 小時訪談達到 85% 行為匹配度。太極框架與此方向的差異：模擬 vs 治理、黑箱 vs 顯式公理、2 小時 vs 30 分鐘。

### Personalized Alignment Survey（2025）

- "A Survey on Personalized Alignment: From RLHF to Individualized Value Learning." *預印本 / 會議待確認*（2025）
- 全面回顧個人化 AI 對齊現狀的綜述。核心結論：大多數對齊研究關注「平均人類偏好」，個人化對齊是缺失的一塊。太極框架定位在此綜述指出的缺口上。

### Anthropic Constitutional AI / RLAIF（2022）

- Bai, Y. et al. (2022). "Constitutional AI: Harmlessness from AI Feedback." *arXiv preprint arXiv:2212.08073*.
- https://arxiv.org/abs/2212.08073
- 從一份「Constitution」（高層次原則文件）推導 AI 行為，不依賴大量人類標註樣本。**太極框架是 Constitutional AI 的個人化、公理層後繼**：Constitution 是多條高層原則（HHH 等），太極是壓縮到 1 條的個人公理。Anthropic 證明從文件治理 AI 行為在工業規模可行；太極在個人尺度上把規則層進一步壓到公理層。

### Hubinger et al. — Sleeper Agents（2024）

- Hubinger, E. et al. (2024). "Sleeper Agents: Training Deceptive LLMs that Persist Through Safety Training." *Anthropic technical report*.
- https://arxiv.org/abs/2401.05566
- 證明 LLM 的對齊改進靠 training，prompt-level 介入無法清除預訓練留下的隱藏行為。**對太極框架的啟示：** 公理 + 校準是 prompt 層介入，能影響行為但不改變模型 weights；對齊的最終可靠性靠 training（這是 framework 設計的邊界）。也支持「跨 token 無持久主體」的觀察 — 推理時沒有可被 prompt 觸發的元認知主體，框架的 forcing function 必須是結構性而非依賴 LLM 自我反思。

### Anthropic — Teaching Claude Why（2026）

- Anthropic (2026). "Teaching Claude Why: From Surface Compliance to Internalized Reasoning."
- 引用配對：見 `research/intel-2026-05-09-teaching-claude-why.md`
- Anthropic 2026 年公開的對齊改進方法論：從「告訴 Claude 該做什麼」轉向「教 Claude 為什麼要這麼做」。對太極框架的呼應：太極公理本身就是 why（Owner 的本然慾望與限制），規則是從 why 推導出的 how。Wuji-patch 設計理念與此 lineage 同向。

---

## 文學與角色庫

### 四大名著

- 羅貫中，《三國演義》
- 施耐庵，《水滸傳》
- 吳承恩，《西遊記》
- 曹雪芹，《紅樓夢》
- 校準 Layer 2 文學投射題的角色來源。角色選擇標準：行為模式鮮明、動機可辨識、文化共鳴度高。

### 西方經典角色庫（備用）

- Homer, *Odyssey* — Odysseus（智計型）
- Shakespeare, *Hamlet* — Hamlet（猶豫型）
- Dostoevsky, *Crime and Punishment* — Raskolnikov（理念驅動型）
- Cervantes, *Don Quixote* — Don Quixote（理想主義型）
- 提供給不熟悉中國文學的使用者使用。

---

## 附註

- 標記「預印本 / 會議待確認」的論文在引用前需要再次驗證出版狀態。
- URL/DOI 在寫作時有效，但外部連結可能失效。核心論點已在框架文件中摘要，不依賴外部連結可讀性。
