# 公版 Wuji 客觀評價 + 自洽真實優化清單

> 評者：brain（2026-05-15）
> 對象：`archive/epochs/wuji-paper-zh.md`（20190 bytes, 2026-05-08 定稿）+ `wuji-saga.md` + `wuji-paper.md`
> 模式：批判性 review，**不護航**；找硬漏洞給 actionable fix
> 自我校核：本 review 須先過 GATE 三問 + 對照 ground truth（paper 原文行號）+ 不確定處標 [未驗證]

---

## A. 強項（保留，不要動）

1. **問題定位精準**：把「LLM 元認知能力不足」識別為跨三紀元的共同根因，與 Kadavath 2022 / Tian 2023 alignment 文獻一致 — §2.4 + §3.3 + §4.1。
2. **不疊規則改循環原語**：避開 EPOCH-1 規則衝突的覆轍，§4.3 設計哲學自洽。
3. **§7 自陳侷限齊全**：單一操作者 / n=18 / 無對照 / 單一提供商 / 封閉資料 — 四項都明示。
4. **§2.5 業界 lineage 對照表 + Constitutional AI 排除註腳**（line 297 自我修正紀錄）：自我修正能力強。
5. **附錄 C 自洽校核機制**：把「貞」原語套用到論文自身，設計優雅。

---

## B. 硬漏洞（11 項，按優先級排序）

### B1. 「補丁閉環」主張過強（§4.3）— **致命**
- 主張：「化必有據 + 貞必接地」雙閉環。
- 實情：兩個補丁都寫在 CLAUDE.md system prompt — 仍是 **prompt-level 提醒**，LLM 是否 inference 時真的查 file:line **無確定性 enforcement**。
- 反例：本身 paper §6.3 + §5.3 承認 fuzzy rate 11%（< 期望 20%）— 證明補丁沒真正觸發「停下查證」的 forcing function。
- **Fix**：§4.3 加表格區分：
  - `prompt-only patches`（PATCH-001 / Wuji Patch 文字寫在 system prompt）— 現況
  - `enforcement patches`（pre-commit hook / lint / tool schema constraint / external verifier）— 缺
  - 把「結構性」claim 弱化為「**位置性結構**（循環原語中的位置）+ 提示性強制（仍依賴 LLM 讀到並 act on it）」

### B2. n=18 樣本的 selection bias 沒處理 — **嚴重**
- §5.1 + §5.3 承認集中於單日（04-13），但**真正的問題是抽樣偏差**：
  - axiom-log 由 agent 自陳記錄 — 「失敗時 agent 也沒元認知 log 失敗」 → 已篩過
  - 61% 正確率裡有 selection bias
- **Fix**：補 §5.4「未記錄錯誤的下界估計」：
  - 拿同期 git log revert/Owner 糾正 instance 數
  - 公式：`real_error_rate ≥ logged_error / (logged_total / coverage_estimate)`
  - 給範圍下界，標明「真實錯誤率上限」

### B3. 「健康自我懷疑水位 ~20%」是 anchor pulled from air — **嚴重**
- §5.3 + §6.3 用 20% 作對比 baseline，**全文無 derivation 也無 citation**。
- 引用的 Kadavath 2022 講 token probability calibration（ECE），不是 metacognition fuzzy rate。
- **Fix**：
  - 改寫：「我們設 20% 為 monitoring anchor target，**無實證基礎 [未驗證]**，僅作監測門檻」
  - 或：給 derivation — 例 Tian 2023 calibration ECE > X% → expected fuzzy rate floor 推導
  - **或最誠實**：拿掉 20%，純做時序趨勢監控（fuzzy rate 隨補丁演進的方向，不設絕對 target）

### B4. §3.2 N(N-1)/2=190 pairs 衝突論證證據不足 — **重要**
- 主張：20 條規則 → 190 對潛在衝突 → 組合爆炸
- 實證：只給 1 個 incident（#47 + #12 刪 3 週標注）
- But-For：少 #47 + 多其他 N 條，衝突真的線性放大嗎？沒有多 incident 證據
- **Fix**：
  - 拿掉「190 pairs」單純計算
  - 補實際觀察到的 incident 列表（即便只 2-3 個）
  - 或弱化：「規則衝突是觀察到的失敗模式之一，組合爆炸是其理論上限，本部署只觀察到 N=X 個 incident」

### B5. 附錄 C 自我校核哲學矛盾 — **重要**
- 全文主論：LLM 無元認知 → 無法自主驗證
- §C：「校核由 agent 自身執行，Owner 以『自洽?』提示時」
- 矛盾：如果 agent 真的不能元認知，§C 校核為何能由 agent 執行有效？
- **Fix**：加 §C.0 前說：
  - 「本校核**依賴外部 artifact**（cross-file diff、grep、memory check），不是 agent 內省」
  - 「Owner trigger『自洽?』是 forcing function 的一個 instance — 校核能力來自**外部觸發 + 外部對照**，非內生」

### B6. 「補丁路徑可能根本不通」沒被挑戰 — **重要**
- §6.3 fuzzy rate 11% 卡住兩補丁 → 結論說「下一補丁很可能落在『感』階段」
- 沒挑戰：補丁路線本身可能根本到達不了 20% 水位
- 缺反向假設章節
- **Fix**：加 §6.4「反向假設」：
  - 「補丁路徑可能根本不通」的可能性
  - 觸發放棄條件：「3 個月後 fuzzy 仍 <15% / 已接地前提仍 OOD failure / 補丁本身被 agent 忽略」3 項任一 → 重估 architectural limit 假設
  - 替代方案：external verifier / RLHF-on-metacognition / 改 base model

### B7. 「結構性 vs 提示性」邊界模糊 — **中重**
- 同 B1 但概念層面：論文混淆「位置性」（在循環的位置）vs「執行強制」（deterministic enforcement）
- **Fix**：§4 開頭加定義：
  - 位置性：補丁位於循環原語的哪個位置（化 / 貞 / 感 / 新）
  - 強制性：補丁是 prompt-level / hook-level / schema-level / external-verifier-level
  - 本論文補丁都在「位置性」維度，強制性僅 prompt-level

### B8. 業界定位表時間軸偏差（§2.5）— **中**
- Devin demo（2024 初）vs GA（2024H2）混淆
- Codex CLI 2024-09 release，放「2024H2-2025」太籠統
- **Fix**：時間範圍改具體月份 — 例：「2024-09 (Codex CLI) ~ 2025-08 (Devin GA)」
- 不影響主論點，但細節 credibility

### B9. 沒有「補丁前 vs 後」決策正確率對比 — **中**
- §5.1 只給 EPOCH-2 後 n=18 數字
- EPOCH-1 後沒對照（雖然樣本不足，但至少 trend 可呈現）
- **Fix**：§5.5 補時序圖（即便只 3-4 個點）：
  - EPOCH-0 → EPOCH-1 → EPOCH-2 → +PATCH-001 → +Wuji
  - 每點標 n + 95% CI
  - 並承認 CI 過大不可推論

### B10. axiom-log protocol 缺失（§7 + 可重現性）— **中**
- 主張可重現性，但沒寫 log 觸發條件
- **Fix**：附錄 D「axiom-log protocol」：
  - schema（CSV 欄位）
  - 觸發條件（每次 Owner 糾正 / 每次 GATE 三問拒絕 / 每天 nudge / ...）
  - 跨 LLM 提供商 prompt format adaptation guide

### B11. 致謝段學術 tone 偏向 — **低（風格選擇）**
- 「致那一句『化的時候，你怎麼知道你的前提是真的？』，產生了 Wuji Patch」太敘事化
- **Fix**：保留 + 加註「以下採非標準學術 tone，反映本框架個人化 origin」 OR 移到 §11「敘事附錄」
- **或不動**：個人風格選擇，不影響學術論點

---

## C. 缺章建議（增補不刪除）

### C1. §6.4 反向假設章節（見 B6）
完整列出補丁路徑放棄條件 + 替代方案

### C2. §8.5 EPOCH-3 預判章節
- 下個失敗模式預判：fuzzy 11% 卡住、感階段 attention drift、單一提供商 lock-in
- 哪些觀察觸發 EPOCH-3 重構

### C3. 附錄 D 可重現性 protocol
axiom-log schema + 觸發條件 + 跨提供商 adaptation

### C4. 附錄 E 數據呈現格式化
- §5.1 表格全部加 95% CI（即便基於 n=18 是大區間）
- 例：「11/18 = 61%, 95% CI [38%, 81%]」— 提示讀者不要 over-interpret

---

## D. 優先級總結（給 Owner 直接做）

| 優先 | 漏洞 | Fix 難度 | 落地後改變 |
|---|---|---|---|
| P0 | B1 prompt-only vs enforcement 區分 | 中（加表+1 段） | 主論點誠實度大幅提升 |
| P0 | B2 selection bias 處理 | 高（要做 git 反推估計） | n=18 結論可信度提升 |
| P0 | B3 20% anchor 標 [未驗證] | 低（改 2 處措辭） | 拔掉 baseless number |
| P1 | B5 §C 自我校核矛盾解釋 | 低（加 §C.0 前說） | 化解自我參照悖論 |
| P1 | B6 反向假設章節 | 中（加 §6.4） | 科學態度體現 |
| P2 | B4 + B8 + B9 + B10 細節考究 | 中 | credibility 增量 |
| P3 | B11 致謝 tone | 低 | 風格選擇 |
| 增補 | C1-C4 | 中 | 完整度 |

---

## E. 一句評語

公版 wuji 是**問題分析 80 分 + 解法主張 60 分 + 學術誠實度 70 分**的工作 — 診斷能力勝過治療能力，補丁所宣稱的「結構性」尚未真正結構化（仍 prompt-level），但已是同類框架中少數明示自我侷限的。優化路線應在「降低主張強度 + 補上 enforcement 層 + 接受補丁路徑可能不通」三條同時走，而非繼續往「下一個補丁」加碼。

---

## F. 本 review 的自洽校核（套用 wuji 自己的「貞」）

- ✅ 所有事實主張對照 paper 行號 / 段落 / 表（line 130-153 wuji patch / line 165-171 n=18 / line 295-304 §C 校核）
- ✅ 不確定處標 [未驗證]（B3 health threshold / B4 multi-incident）
- ✅ 不引用未讀的外部來源（Kadavath 2022 + Tian 2023 僅指出 paper 引用方式不當，未編造其結論）
- ⚠️ 本 review 由 agent 撰寫，依 wuji 自身主論點「LLM 無元認知」— 此 review 的判斷力上限同 paper §C 一樣依賴外部對照（paper 原文 + memory + Owner 校準），非內生。
- 📋 Owner 應外部校驗 B2 / B6 / B9 三條的可行性後再決定是否動。
