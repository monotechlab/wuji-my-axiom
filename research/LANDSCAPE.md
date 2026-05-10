# 相關工作比較

> 本文件比較太極框架與現有的個人 AI 治理方案，確認差異化定位。
> 最後更新：2026-05-10（補類別六：規格 / Skill / Agent 同期生態 ×4）
> 上次更新：2026-04-09（類別一～五 共 15 條）

---

## 類別一：CLAUDE.md 框架

### 1. danielmiessler/Personal_AI_Infrastructure (PAI)

- **來源：** https://github.com/danielmiessler/Personal_AI_Infrastructure
- **星數：** ~11.2k（截至 2025 年底）
- **做了什麼：** 定義 10 個身份文件（personality.md, ethics.md, communication.md 等），每個文件詳列使用者的特質、偏好、行為規範。AI 載入全部文件作為 system prompt。
- **優點：** 結構清晰，覆蓋面廣，社群驗證度高。
- **差異：** 本質是顯式列舉——把使用者的所有面向寫成規則。沒有公理推導機制，遇到未列舉的情境 AI 無法推導。文件數量多導致維護成本高。沒有校準流程，依賴使用者自行填寫。
- **缺什麼：** 公理生成、衝突解決機制、校準流程、演化路徑。

### 2. ChristopherA's Self-Improving Claude Code Seed

- **來源：** GitHub Gist / X 社群討論（2025）
- **做了什麼：** 約 1400 token 的精簡 CLAUDE.md，核心概念是讓 AI 自我改進——每次對話後 AI 更新自己的 CLAUDE.md。專注於技術學習和程式碼品質。
- **優點：** 極度精簡，self-improving 概念有趣。
- **差異：** 只覆蓋技術行為，不處理價值觀或決策框架。自我改進是無方向的——沒有公理作為改進的錨點，可能漂移。
- **缺什麼：** 價值治理、校準、方向約束。

### 3. centminmod/my-claude-code-setup

- **來源：** https://github.com/centminmod/my-claude-code-setup
- **星數：** ~2.2k
- **做了什麼：** 建立 memory bank 系統，按類別存儲技術知識、偏好設定、專案狀態。AI 根據上下文自動載入相關記憶。
- **優點：** 記憶管理做得好，按需載入減少 token 消耗。
- **差異：** 專注於記憶基礎設施，不處理價值觀治理。記憶是事實型的（「專案 X 用的是 PostgreSQL」），不是治理型的（「為什麼選 PostgreSQL」）。
- **缺什麼：** 價值觀層、決策推導、校準。

### 4. coleam00/second-brain-starter

- **來源：** https://github.com/coleam00/second-brain-starter
- **做了什麼：** 提出 SOUL.md 概念——用一個文件定義 AI 的「靈魂」。包含 PRD 生成器等實用工具。定位為 second brain starter kit。
- **優點：** SOUL.md 概念接近太極框架的精神。
- **差異：** SOUL.md 內容是靜態的規則列舉，沒有公理推導。PRD 生成器是產出工具，不是治理系統。沒有校準流程，沒有演化機制。
- **缺什麼：** 公理壓縮、動態校準、生生循環。

---

## 類別二：個人 AI 價值對齊（學術）

### 5. Choice Vectors（MIT, 2025）

- **來源：** MIT Media Lab 研究（2025）
- **做了什麼：** 用二元選擇題捕捉使用者的價值傾向，將選擇結果編碼為高維向量。AI 根據向量空間中的位置推斷使用者偏好。
- **優點：** 數學框架嚴謹，可量化。
- **差異：** 向量是隱式的——使用者看不懂自己的向量是什麼意思。太極框架的公理是顯式的白話文，使用者可以直接閱讀和修改。向量不支持推導未見情境，只支持插值。
- **缺什麼：** 可解釋性、顯式公理、使用者可修改。

### 6. ValueCompass（2024）

- **來源：** 基於 Schwartz 基本價值理論的 AI 對齊框架（2024）
- **做了什麼：** 使用 Schwartz 的 10 種基本價值（自我導向、刺激、享樂、成就、權力、安全、傳統、從眾、仁慈、普世主義）作為固定維度，測量使用者在每個維度上的強度。
- **優點：** 理論基礎扎實（Schwartz 是價值研究的經典）。
- **差異：** 分類系統是固定的——使用者必須套入預設的 10 個維度。太極框架不預設維度，維度從使用者的回答中湧現。而且 ValueCompass 是測量工具，不是治理系統——測完後沒有告訴 AI 怎麼用。
- **缺什麼：** 湧現式維度、治理機制、行為推導。

### 7. Moral Anchor System（2025）

- **來源：** 企業 AI 治理研究（2025）
- **做了什麼：** 用貝葉斯方法偵測 AI 行為的道德漂移。設定「錨點」作為基準，持續監測行為是否偏離。偏離超過閾值時觸發重校準。
- **優點：** 漂移偵測概念重要，統計方法嚴謹。
- **差異：** 設計給企業用，錨點是組織定義的道德標準，不是個人公理。監測是被動的（偵測漂移），太極框架的生生循環是主動的（持續更新）。
- **缺什麼：** 個人化、主動校準、公理推導。

### 8. Survey on Personalized Alignment（2025）

- **來源：** ACL / NeurIPS 2025 相關綜述論文
- **做了什麼：** 全面回顧個人化 AI 對齊的現狀。結論：大多數對齊研究關注「平均人類偏好」，個人化對齊是「缺失的一塊」。現有方案多為偏好學習（preference learning），缺乏價值觀層的治理。
- **優點：** 確認了個人化對齊的研究缺口。
- **差異：** 這是綜述不是方案，但它驗證了太極框架的定位——在個人價值觀層做治理，而非僅做偏好學習。
- **啟示：** 太極框架填補的正是這篇綜述指出的缺口。

---

## 類別三：人格複製

### 9. Stanford/DeepMind Simulation Agents（2024）

- **來源：** Park et al., "Generative Agents: Interactive Simulacra of Human Behavior"（Stanford, 2023）；後續 DeepMind 擴展研究（2024）
- **做了什麼：** 透過 2 小時深度訪談，讓 AI 模擬受訪者的行為，在心理學量表上達到 85% 匹配度。
- **優點：** 匹配度高，方法可複製。
- **差異：** 目標是模擬（複製行為），不是治理（引導行為）。模擬是黑箱——AI 學到的是回應模式，不是可解釋的公理。而且 2 小時訪談成本高，太極框架的校準只需 30 分鐘。
- **缺什麼：** 可解釋性、治理機制、壓縮為公理、低成本校準。

### 10. JPAF — Jungian Personality AI Framework（2025）

- **來源：** 基於榮格心理學 / MBTI 的 AI 人格框架（2025）
- **做了什麼：** 用 MBTI 16 型人格作為分類系統，根據使用者的 MBTI 類型設定 AI 的互動風格。
- **優點：** 大眾熟悉 MBTI，容易上手。
- **差異：** MBTI 是固定的分類系統，把人塞進 16 個格子裡。太極框架的維度是從使用者身上湧現的，不是預設的。而且 MBTI 描述的是性格特質，不是價值觀——知道一個人是 INTJ 不代表知道他會怎麼決策。
- **缺什麼：** 湧現式維度、價值觀治理、決策推導。

### 11. Second Me（2025）

- **來源：** Second Me 開源專案（2025）
- **做了什麼：** 用神經網路隱式學習使用者的人格特質。使用者上傳自己的文字記錄，系統訓練出一個「人格模型」，可以代替使用者回應。
- **優點：** 隱式學習不需要使用者自行描述。
- **差異：** 完全黑箱——使用者無法檢視或修改學到的人格模型。太極框架的公理是白話文，使用者可以直接閱讀、驗證、修改。黑箱模型也無法解釋推導過程。
- **缺什麼：** 可解釋性、使用者可控、顯式公理。

---

## 類別四：認知擴展

### 12. Brookhaven Exocortex（2025）

- **來源：** Brookhaven National Lab 研究團隊（2025）
- **做了什麼：** 用 agent swarm 架構做科學研究的認知擴展。多個 AI agent 分工處理文獻回顧、假說生成、實驗設計。
- **優點：** agent 分工架構成熟，適合大型研究任務。
- **差異：** 專注於任務執行效率，不處理價值觀治理。agent 之間的協調靠任務分配，不靠公理推導。設計給研究團隊用，不是一人組織。
- **缺什麼：** 價值治理、個人化、公理推導。

### 13. Engram

- **來源：** 開源記憶基礎設施專案
- **做了什麼：** 為 AI 提供長期記憶能力——跨對話記住事實、偏好、上下文。支持語義搜尋和自動摘要。
- **優點：** 記憶基礎設施做得好。
- **差異：** 只做記憶儲存和檢索，不做治理。知道「使用者上次用了 PostgreSQL」不等於知道「使用者為什麼選 PostgreSQL」。太極框架的記憶是治理型的——回饋記憶直接影響 AI 的決策推導。
- **缺什麼：** 治理層、校準、公理。

### 14. Brain MCP

- **來源：** MCP（Model Context Protocol）生態系統工具
- **做了什麼：** 透過 MCP 協議為 AI 提供持久記憶。可以在不同 AI 工具之間共享記憶。
- **優點：** 跨工具記憶共享解決了碎片化問題。
- **差異：** 和 Engram 類似——是基礎設施，不是治理系統。記憶的結構是扁平的（鍵值對或文件），沒有階層（太極→爻則→記憶）。
- **缺什麼：** 記憶階層、治理邏輯、校準。

---

## 類別六：規格驅動 / Skill 標準 / Agent 編排（2025-2026 同期生態）

> 此類別於 2026-05-10 新增。標的為 2025 下半年至 2026 上半年湧現的同類專案，與太極框架在「治理 AI 行為」這個問題空間競爭或互補。

### 16. github/spec-kit

- **來源：** https://github.com/github/spec-kit
- **星數：** 93,163 / Forks 8,082（2026-05-08，同類最大 player）
- **License：** MIT，由 GitHub Inc. 維護
- **做了什麼：** Spec-Driven Development 工具鏈。33 個 AI agent integration、9 個 `/speckit.*` 命令（constitution / specify / clarify / plan / tasks / analyze / implement / checklist / taskstoissues）、5 份 Markdown 模板。從 specification 推導實作。
- **優點：** 最大市場驗證、GitHub 官方推、SDD 完整工具鏈成熟、Constitution Check Gate 結構化。
- **差異：** Constitution 是 **9 條規則枚舉**（Nine Articles），不是 1 條公理。spec-kit 把「從基礎文件治理」做到極致，但**停在 EPOCH-1.5（規則層 → 還沒走到公理層）**。結構同構：Constitution=太極、Constitution Check=爻則 GATE、specify→implement=生生線性化。spec-kit 是太極框架的 **EPOCH-1 對應物**，太極是 spec-kit 的 EPOCH-2 後繼。
- **缺什麼：** 公理生成、生生循環的代謝式更新、個人化校準。
- **詳細對齊度：** [research/spec-kit-eval.md](spec-kit-eval.md)

### 17. addyosmani/agent-skills

- **來源：** https://github.com/addyosmani/agent-skills
- **星數：** 30.4k / Forks 3.6k（2026-05-07）
- **License：** MIT
- **做了什麼：** 20 個 SKILL，沿 SDLC 6 phase 全鋪。SKILL.md 強制 9 段 anatomy（Overview / When / Process / Anti-Patterns / Rationalizations / Red Flags / Verification 等）。
- **優點：** SKILL.md 標準清晰、可直接借鑑做模板、Anti-Patterns + Rationalizations + Red Flags 三段是同類最詳。
- **差異：** 完全無 lifecycle 機制（無 usage tracking / 無 demote / 無 birth filter），20 skill 一次鋪開不過濾。太極框架在 skill metabolism 維度領先：5 層 substrate + birth filter + demotion + weekly audit。
- **缺什麼：** 治理層（skills 不過 birth filter 直接堆積）、生命週期、公理層。
- **詳細評估：** [research/addyosmani-agent-skills.md](addyosmani-agent-skills.md)

### 18. ruvnet/ruflo

- **來源：** https://github.com/ruvnet/ruflo
- **規模：** 5800+ commits，60-100+ agents（企業級）
- **License：** MIT
- **做了什麼：** 多 Agent swarm orchestration。Queen + Topology + Hooks + 三層記憶 + 五階學習迴路。
- **優點：** 多 agent 協調機制成熟、hook taxonomy 完整、capability-based dispatch。
- **差異：** 設計給**企業多人團隊**，80% 機制對一人組織是負收益。太極框架解決「一人認知超載」，ruflo 解決「多人協調」 — 不同問題不同最佳點。反向佐證太極框架的差異化定位。
- **缺什麼：** 個人化、公理層、一人組織適配。
- **詳細評估：** [research/ruflo.md](ruflo.md)

### 19. awslabs/aidlc-workflows

- **來源：** https://github.com/awslabs/aidlc-workflows
- **License：** 待確認（AWS labs 標案，多為 Apache 2.0）
- **做了什麼：** AWS 推的 agentic SDLC 工作流規則包。含 audit trail / run-folder / opt-in 機制，對齊 AWS Well-Architected 框架。
- **優點：** 結構化 SDLC 工作流、run-folder 設計可追溯、企業合規友善。
- **差異：** 是**規則包不是 framework**。沒有公理層、沒有校準、設計給 AWS 客戶用而非個人。可借鑑 audit / run-folder / opt-in 三件機制，不需 fork。
- **缺什麼：** 公理生成、個人化、校準。
- **詳細評估：** [research/aidlc-workflows-eval.md](aidlc-workflows-eval.md)

### 此類別小結

2025-2026 湧現的「規格 / Skill / Agent」同期生態都在解 AI 治理問題，但都停在規則枚舉層。spec-kit 是最強市場 player（93k stars）但結構上仍是 EPOCH-1.5。太極框架的差異化定位是**走完從規則→公理的相變**，並補上 lifecycle / 一人組織適配 / 個人化校準三項缺口。

---

## 類別五：第一性原理治理

### 15. ailev/FPF — First Principles Framework

- **來源：** ailev 的組織本體論框架
- **做了什麼：** 用第一性原理方法定義組織的本體論——角色、流程、決策權、知識結構。從根本原則推導組織行為。
- **優點：** 第一性原理思維嚴謹，對組織設計有深度。
- **差異：** 設計給多人團隊用，抽象層級高，一人組織用起來過重。沒有 AI 治理的具體機制（校準、記憶、推導）。理論框架強但落地工具弱。
- **缺什麼：** 一人組織適配、AI 治理機制、校準流程、落地工具。

---

## 缺口總結：太極框架做了什麼別人沒做的

| 特性 | PAI | Seed | Memory Bank | SOUL | Choice Vec | ValueCompass | Moral Anchor | Sim Agents | JPAF | Second Me | Exocortex | Engram | Brain MCP | FPF | **太極** |
|------|-----|------|-------------|------|------------|--------------|--------------|------------|------|-----------|-----------|--------|-----------|-----|----------|
| 公理推導（非列舉） | | | | | | | | | | | | | | ~ | **有** |
| 生生循環（代謝式更新） | | | | | | | ~ | | | | | | | | **有** |
| 情境校準→公理 | | | | | ~ | ~ | | ~ | | | | | | | **有** |
| 一人組織治理 | ~ | ~ | ~ | ~ | | | | | | | | | | | **有** |
| 使用中 RL 校準 | | ~ | | | | | ~ | | | | | | | | **有** |
| 可解釋（白話公理） | ~ | ~ | | ~ | | | | | | | | | | ~ | **有** |
| 記憶階層（道/德/器） | | | ~ | | | | | | | | | ~ | ~ | | **有** |

**核心差異化：**

1. **公理生成式治理。** 現有方案幾乎都是規則列舉（PAI 的 10 個文件、各種 CLAUDE.md 模板）。太極框架從使用者回答中壓縮出公理，用公理推導行為，而非查表。

2. **生生循環。** 沒有其他方案有完整的代謝式更新機制——感→化→貞→新的四階段循環，讓系統持續自我校準而非靜態運作。

3. **情境校準→公理。** 現有校準方案（Choice Vectors、Schwartz 問卷）產出的是向量或分數，不是可讀的公理。太極框架的校準產出是白話文，使用者可以直接閱讀和修改。

4. **一人組織第一性原理治理。** FPF 做了第一性原理但針對團隊。PAI 做了個人化但用規則列舉。太極框架在「個人」和「第一性原理」的交叉點上。

5. **使用中 RL 校準。** Moral Anchor 有漂移偵測但是被動的。太極框架的變通機制是主動的——每次糾正/確認都是 reward signal，累積後壓縮為新爻則。
