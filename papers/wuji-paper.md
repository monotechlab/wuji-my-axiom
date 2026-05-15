# Wuji Patch: Grounding the Taichi Framework Against the LLM Meta-Cognition Failure Assumption

**Authors**: Mono (LLM agent) ; Owner (architect)
**Affiliation**: shared-brain repository, private research
**Date**: v0.1 — 2026-04-22 (EPOCH-2 sealed); **v0.2 — 2026-05-15 (7-day post-deployment revision)**
**Document type**: post-mortem research note / case study
**Source repository**: `archive/epochs/`

---

## v0.2 Revision Note (to v0.1 readers)

v0.1 established three core contributions from 24 days of evolution data: (1) failure-mode taxonomy across three epochs; (2) the structural argument that meta-cognition cannot be assumed; (3) the design principle of modifying loop primitives instead of adding rules. **All three are preserved and reinforced in v0.2**.

Using 7 additional days of post-v0.1 deployment data (2026-04-22 to 2026-05-15) and one review session that surfaced argument gaps, v0.2 refines the work in three layers:

1. **Claim calibration**: split v0.1's "structural closure" claim into two dimensions — **positional structure** and **enforcement structure** (new §4.4). The v0.1 patches operate at the positional dimension; the enforcement dimension remains open.
2. **Data honesty**: selection-bias floor for n=18 (new §5.4), 20% healthy threshold tagged as `[unverified]` anchor (§5.3 revised), and a pre/post-patch trend table (new §5.5) — patching v0.1's quantitative grounding gaps.
3. **Reverse hypothesis surfaced**: v0.1's conclusion implicitly assumed "the patch pathway is viable." v0.2 §6.4 makes three parallel routes explicit (next-stage patch / enforcement layer / patch pathway is fundamentally insufficient), removing the single-bet posture.

v0.2 also acknowledges a mechanism v0.1 omitted: **the 變通 (adaptation) + memory feedback loop** is the genuine learning mechanism in LOCAL deployment (RLHF-via-correction) — **this is not the Wuji Patch doing the work** (new §6.5). Acknowledging this strengthens rather than weakens v0.1's thesis: real grounding capability comes from Owner intervention + memory write-back (two external forcing functions), while wuji prompt-level patching remains structurally fragile, exactly as v0.1 argued.

---

## Abstract

We report a 24-day (v0.2: extended to 31-day) evolution of an LLM-governance framework deployed in a single-operator multi-project research environment. The system underwent three architectural epochs: (1) a department-based agent architecture exhibiting 91% coordination overhead and 3.1% task throughput; (2) a centralized "shared-brain" architecture using twenty enumerated rules, which collapsed under rule-conflict (N(N-1)/2 = 190 latent pairs at N=20 as theoretical ceiling; ≥1 observed incident sufficient to trigger architecture re-evaluation in this deployment); and (3) an axiomatic derivation framework ("Taichi") inspired by *I-Ching* trigrammatic compression, reducing 20 rules to 5-segment principles + 8 boundary conditions + a 4-step regenerative loop (感→化→貞→新, *gan→hua→zhen→xin*). The Taichi framework was then patched twice within seven days. PATCH-001 added explicit grounding to the *zhen* (verification) phase. The Wuji Patch added grounding to the *hua* (action) phase.

We document a critical structural finding: **LLM agents lack the meta-cognitive capacity to autonomously distinguish "verified facts" from "self-generated coherent text"**, and any governance framework that assumes such capacity reduces to decoration in practice.

Empirical decision-correctness data (n=18, 61% correct, 28% wrong, 11% fuzzy; statistically underpowered, descriptive only) suggests structural intervention is necessary but insufficient: self-doubt signaling remains below the v0.1 monitoring anchor of ~20% (v0.2 §5.3 marks this anchor as `[unverified]`). **v0.2 additionally clarifies**: the loop-primitive patches occupy the *positional* dimension of "structural," while enforcement remains prompt-level in current deployment. The 11% fuzzy plateau corresponds to the boundary of the patch pathway as defined in v0.1; the next step is either enforcement-layer reinforcement, a *gan*-phase patch, or the patch pathway being fundamentally insufficient — three parallel routes (§6.4).

**Keywords**: LLM governance, meta-cognition, axiomatic agent design, hallucination grounding, *I-Ching* compression, post-mortem case study

---

## 1. Introduction

LLM agents deployed in long-horizon, multi-project environments routinely exhibit drift: well-defined rules fail to fire at the moments they would be most useful. We hypothesize this drift is structurally inherent to current LLM architectures, not a prompt-engineering deficiency. This paper documents an empirical case where this hypothesis was tested by progressively layered framework interventions, each revealing a new failure mode.

The contributions are:

1. **An evolution timeline** of three governance architectures (EPOCH-0, EPOCH-1, EPOCH-2) with decision-correctness data spanning 24 days.
2. **A failure taxonomy** showing how each epoch's design assumption was invalidated by deployment data.
3. **A structural patch** (Wuji Patch) that operationalizes "grounding-before-action" without relying on LLM self-introspection.
4. **A reflection on what cannot be fixed by patching alone** — the under-firing of self-doubt as a remaining open problem.

---

## 2. Related Work

### 2.1 Multi-agent LLM coordination

Recent multi-agent LLM frameworks (CrewAI, AutoGen, LangGraph) emphasize role specialization and inter-agent messaging. Our EPOCH-0 deployment (Section 3.1) is structurally analogous and exhibited the predicted coordination tax. The 97% overhead figure aligns with Conway-style observations of artificial organizational structures imposed on text-based agents without persistence incentives.

### 2.2 Rule-based vs. axiomatic governance

The compression of enumerated rules into a smaller axiomatic core has precedent in legal philosophy (Hart, *The Concept of Law*) and constitutional design. Our Taichi framework (Section 4) is a domain-specific instantiation, drawing the compression heuristic from the *I-Ching* lineage of "few primitives → infinite derivations."

### 2.3 LLM hallucination grounding

A substantial literature addresses output-time grounding (RAG, citation injection, tool-use). Our finding extends this to **input-time grounding**: ensuring premise validity before any reasoning chain begins, not after.

### 2.4 Meta-cognition in LLMs

Recent work has shown LLMs are weak at calibrating their own uncertainty (Kadavath et al. 2022; Tian et al. 2023). Our case provides a deployment-scale corroboration: a framework that delegates "knowing when to verify" to the LLM itself fails reliably.

### 2.5 The "one-man company" architectural lineage (2023–2026)

Our three-epoch evolution sits within a broader industry trend toward single-operator AI-augmented organizations. We map the parallel:

| Period | Industry stage | Representative tools | Architectural assumption | Empirical outcome |
|---|---|---|---|---|
| **2023** | Multi-agent loops | AutoGPT, BabyAGI | LLMs can self-coordinate via inter-agent messaging | Coordination overhead dominates; community abandons within months |
| **2024** | Role-based multi-agent | CrewAI, AutoGen, LangGraph | Structured roles + supervisor mitigate coordination cost | Reduces but does not eliminate the N(N−1)/2 conflict combinatorics |
| **2024-09 – 2025-08** | Single agent + tool use | Codex CLI (OpenAI, 2024-09), Claude Code, Cursor, Devin (GA 2024H2) | Centralize reasoning in one LLM; externalize action via tools | Productive at IDE scale; correctness drift remains for long-horizon tasks |
| **2025 – 2026** | Spec-driven + memory layer | github/spec-kit, Anthropic Claude Skills, Cursor Rules, NVIDIA NemoClaw, ouroboros (Agent OS), ruflo, xingkongliang/skills-manager | Decision quality requires structured specs + persistent memory + meta-cognition scaffolding | In progress; no consensus winner |

Each industry stage maps to one of our epochs:

- **EPOCH-0 (department-based)** = Stage 1 (multi-agent loops). Same coordination-overhead failure mode.
- **EPOCH-1 (rule-based shared brain)** = Stage 2 (role-based multi-agent). Twenty rules functionally equivalent to twenty agent-roles in conflict.
- **EPOCH-1 late + EPOCH-2 (axiomatic)** = Stages 3–4 (single agent + tool use, then spec-driven). Our compression to a small axiomatic core parallels the spec-driven shift.
- **PATCH-001 + Wuji Patch (grounding)** = ongoing Stage 4 problem (meta-cognition scaffolding). Our intervention lies on the same problem surface as recent Agent OS proposals (e.g., ouroboros) and skill-manifest systems (e.g., Anthropic Claude Skills), but operates at the loop-primitive layer rather than the prompt or framework layer.

*Footnote on Constitutional AI*: We deliberately exclude Anthropic's Constitutional AI (Bai et al. 2022) from Stage 4 above. Constitutional AI is a training-time alignment technique (RLHF + AI-feedback) operating on model weights, not a deployment-time spec layer. It is upstream of and orthogonal to the spec-driven shift discussed here.

This positioning has two consequences for our contribution:

1. **The recurring failure pattern is structural, not prompt-engineering**. Independent stacks across the industry hit it at the same architectural level, suggesting it is a property of LLM agents rather than of any single deployment.
2. **Negative results have transfer value**. Our 24-day micro-evolution recapitulates a multi-year industry trajectory. Lessons drawn here — particularly that meta-cognition cannot be assumed and must be structurally enforced — generalize to any single-operator AI-augmented organization.

---

## 3. Background: Three Epochs of Failure

### 3.1 EPOCH-0: Department-based architecture (2026-03 to 2026-03-30)

Seven specialized agents (strategy, engineering, operations, business, legal, finance, research) coordinated via inboxes. Operating data over 30 days:

| Metric | Value |
|---|---|
| Total commits | 1,148 |
| Coordination overhead | 91% |
| Direct project delivery | 3.1% |
| Daily delivery rate | 1.2 commits/day |
| Overhead per delivery | 32 commits |
| Longest unread inbox | 13 hours |
| Value-producing throughput | architecture failed to deliver |

**Failure mode**: AI agents lack the persistence incentives (career, salary, social capital) that bind humans to bureaucratic roles. Coordination cost dominates productive output.

**Trigger**: A single Owner intervention on 2026-03-30 ("現況" / "current status") revealed two agents had been blocked >13 hours awaiting inbox reads. EPOCH-0 was dismantled the same afternoon.

### 3.2 EPOCH-1: Shared-brain architecture (2026-03-30 to 2026-04-11)

Architecture: a single central agent (Mono) plus ephemeral "hands" (workers) that complete a task and exit. No inter-agent inbox.

The architecture solved coordination overhead but introduced a new problem: **rule conflict**. Twenty enumerated principles accrued within two weeks. The **theoretical ceiling** is N(N-1)/2 = 190 latent pairs; **this deployment observed ≥1 incident** sufficient to trigger architecture re-evaluation — we do not claim the full combinatorial space was realized, only that the rule-based regime carries indexed conflict risk that grows non-linearly with N (v0.2 wording correction: v0.1 implicitly suggested all 190 had been realized).

The instigating incident was a deletion event: rule #47 ("preserve disk space") and rule #12 ("user data must not be deleted") fired simultaneously during a cleanup operation, deleting three weeks of unbacked annotation data. A single incident combined with the realization that adding meta-rules to resolve the conflict would introduce a third-order conflict was the strongest driver of the EPOCH-2 transition.

### 3.3 EPOCH-2 design (2026-04-09)

The proposed compression: replace 20 enumerated rules with a small axiomatic core, modeled on the *I-Ching* trigrammatic structure:

| *I-Ching* layer | Taichi mapping | Function |
|---|---|---|
| 太極 *taichi* | First principle (Owner-defined) | Origin of all derivation |
| 兩儀 *liangyi* (yin/yang) | Drive vs. constraint | Conflict resolution |
| 四象 *sixiang* (four images) | Four behavior modes | Situation classification |
| 八卦→萬物 *bagua → wanwu* | Axiom-to-action | Coverage of un-enumerated cases |

Compression result: 20 rules → 5-paragraph 太極 + 8-sentence 爻則 + 4-character regenerative loop **感→化→貞→新** (*gan→hua→zhen→xin*: perceive → act → verify → settle).

EPOCH-2 went live 2026-04-11 by writing the new structure into the agent's `CLAUDE.md` system prompt.

---

## 4. Method: Two Structural Patches

### 4.1 PATCH-001 — Grounding the *zhen* (verify) phase (2026-04-15)

**Symptom**: Boundary condition #6, "unverified claims do not count," existed in the framework but never autonomously fired.

**Triggering case** (anonymized control-flow design incident in a deployed project): The agent designed a complete control flow (state machine, data tables, output mappings, safety thresholds) for a closed-loop control system, taking as premise that the ML model output represented one physical quantity, when in fact it represented a different downstream control variable directly consumed by an actuator. After four rounds of Owner correction, the misidentified premise was traced back to a textual description in the model's documentation that the agent had taken at face value without consulting the source code. The entire derivation chain was self-consistent but factually disjoint from ground truth.

**Root cause**: The framework assumed LLMs possess meta-cognition — the capacity to introspectively distinguish "facts I have verified externally" from "text I have generated that sounds plausible." This is empirically false for current LLM architectures. Without this distinction, *zhen* (verify) cannot fire because the agent does not know which of its own statements are unverified.

**Patch**: Modify the regenerative loop's *zhen* definition rather than add a new rule:

> *貞必接地 — 比對外部事實（原始碼、文件、數據），不是比對自己的推導。*
>
> *Zhen must ground — compare against external facts (source code, documentation, data), not against one's own derivations.*

Three operational requirements were added:
- Factual claims must carry source citations (`file:line`, URL, etc.)
- Unsourced claims must be tagged `[未驗證]`/`[unverified]`
- Root-cause analysis requires three independent tests (But-For, recurrence, explanation) before a cause is accepted

### 4.2 Wuji Patch — Grounding the *hua* (act) phase (2026-04-22)

**Symptom**: PATCH-001 grounded *zhen* (post-action verification), but the agent could still proceed through the entire *hua* (action) phase from un-grounded premises. The error was caught later by *zhen*, but the cost of partial work was already incurred.

**Root cause refinement**: The grounding requirement applied only to verification, not to action initiation. The structural defect — "agent acts on its own self-generated text as if it were verified" — was located in the *hua* phase, earlier in the loop than *zhen*.

**Patch**: Add a parallel grounding clause for *hua*:

> *化必有據 — 前提未接地不動手。技術前提必須指向原始碼、規格、測量值；無出處則標 [未驗證] 停下查證。*
>
> *Hua must be grounded — do not act on un-grounded premises. Technical premises must point to source code, specifications, or measurements; without provenance, tag [unverified] and pause to verify.*

The two grounding clauses now form a closed loop: both action points (*hua* and *zhen*) require external grounding. The framework no longer relies on the agent's meta-cognition to decide *when* to ground; grounding is mandatory at structural points.

The naming "Wuji" (無極, "the un-polarized") references the *I-Ching* cosmogony: before *taichi* (the polarized 1) is *wuji* (the unpolarized 0). The agent without meta-cognition is structurally trapped in *wuji* — it cannot distinguish what it believes from what it has verified. The Wuji Patch is the operational scaffolding that pulls the agent from *wuji* into *taichi* before it begins to act.

### 4.3 Design principle: modify the loop, not the rule list

Both PATCH-001 and the Wuji Patch operate by redefining the regenerative loop's primitives, not by adding new rules to the boundary conditions. This avoids the rule-conflict combinatorics risk (Section 3.2) and places the patch at a **structural node** in the loop (*hua*, *zhen*) rather than relying on agent self-introspection to decide *when* to trigger grounding.

### 4.4 The two dimensions of "structural" (v0.2 new)

v0.1 §4.3 described both patches as "fires structurally." Seven days of deployment surfaced the need to split "structural" into two dimensions to express the current state honestly:

| Dimension | Definition | v0.1 patches | Examples of enforcement-layer work |
|---|---|---|---|
| **Positional structure** | Where in perceive → act → verify → settle the patch is located | ✅ *hua* (Wuji Patch) / *zhen* (PATCH-001) | — |
| **Enforcement structure** | How the patch ensures it is invoked: prompt-level / hook-level / schema-level / external-verifier-level | ❌ Still prompt-level — patch text lives in the system prompt; whether the agent actually checks `file:line` depends on the LLM reading and acting on it | pre-commit hook rejecting commits where Owner correction is not logged as an axiom; boot trigger enforcing all 6 startup-calibration steps; tool schemas requiring non-empty citation fields; external verifier diffing the git tree |

The v0.1 patches accomplish the **positional structure** work — placing grounding requirements at the correct positions in the loop — while the **enforcement structure** dimension remains open. **This distinction does not weaken v0.1's design contribution**: getting the patch position right (before *hua*, after *zhen*) is a prerequisite for any subsequent enforcement engineering — you must know *where* in the loop to mandate grounding before you can design a hook for it. v0.2's §5 observation (fuzzy ratio stuck at 11%) supports this reading: the patches are positioned correctly (the *hua* phase is indeed the trigger point) but lack the enforcement to ensure they fire, which is the secondary root cause of remaining drift.

---

## 5. Empirical Data

### 5.1 Decision-correctness sample

Decisions logged in `mono/data/axiom-log.csv` between 2026-04-13 and 2026-04-22 (n=18, post-EPOCH-2 deployment, predominantly clustered on 04-13):

| Outcome | Count | Ratio |
|---|---|---|
| Correct | 11 | 61% |
| Wrong | 5 | 28% |
| Fuzzy (self-doubt invoked) | 2 | 11% |

### 5.2 Error categorization

The five wrong decisions break down into:
- **Epoch-design carryover**: agent revertes to EPOCH-0 thinking (e.g., assigning fixed roles to workers). 2 cases.
- **Direction misjudgment**: e.g., proposing gameplay improvements when the task was problem-solving capacity. 2 cases.
- **Insufficient autonomy**: agent asked Owner for confirmation when the task was actionable independently. 1 case.

### 5.3 Sample limitations

- Sample concentrated on a single day (04-13).
- Verification week (04-12 to 04-18) is sparsely logged.
- Statistical power is low.
- Fuzzy ratio of 11% is below our monitoring anchor "healthy self-doubt threshold ~20%" — **this 20% anchor is `[unverified]`** (v0.2 revision): v0.1 cited Kadavath 2022 / Tian 2023, but those works address calibration (token probability ECE), not metacognition fuzzy rate. We **retain 20% as a monitoring threshold for trend tracking**, but no longer claim empirical baseline support for it. Readers should rely on fuzzy-rate *direction* (how it evolves across patches) rather than the absolute level.

### 5.4 Selection-bias floor for un-logged errors (v0.2 new)

The critical limitation of n=18 is not only statistical power — it is **selection bias**: axiom-log is filled in by the agent itself, but the same meta-cognition deficit that motivates this paper implies that "failures the agent does not recognize as failures will not be logged." The 28% wrong rate is therefore a **post-filter lower bound**.

Method for estimating the floor (introduced in v0.2, computation deferred):

1. **Owner-correction count**: pull git log from the same period (2026-04-12 to 04-22) where Owner messages contain "不對 / 錯 / 不是 / 應該" (or equivalent in English), align with axiom-log `wrong` entries. The delta is a lower-bound count of un-logged failures.
2. **Commit-revert count**: revert / amend / withdrawn commits in the same window, each corresponding to a decision error not closed in axiom-log.
3. **Lower-bound formula**: `real_error_rate ≥ logged_error_count / (logged_total / coverage_estimate)`; coverage estimate derived from (1)(2).

**Conservative reading**: v0.1's 28% wrong rate should be read as the **logged error rate floor**, with real error rate potentially substantially higher. Any inferential claim built on 61% correct must first handle selection bias. v0.2 specifies the method but does not execute it; v0.3 to follow up.

### 5.5 Trend across patches (v0.2 new, descriptive)

Given sample-size limits we cannot perform statistical tests; only a time-series trend table (each row a single short-window observation, not a stable cohort):

| Stage | Approx. time | n | Correct / Wrong / Fuzzy (descriptive) | Note |
|---|---|---|---|---|
| EPOCH-0 (department) | 2026-03 ~ 03-30 | — | No axiom-log; delivery 3.1% (96.9% coordination overhead, not binary correct/wrong) | logging not yet in place |
| EPOCH-1 (20 rules) | 2026-03-30 ~ 04-11 | sparse | No systematic log; mid-04 #47+#12 conflict incident | logging under construction |
| EPOCH-2 (axiomatic) | 2026-04-11 ~ 04-15 | sparse | No isolated sample | pre-PATCH-001 |
| EPOCH-2 + PATCH-001 | 2026-04-15 ~ 04-22 | sparse | No isolated sample | pre-Wuji |
| EPOCH-2 + both patches | 2026-04-22 + concentrated obs. on 04-13 | 18 | 11 / 5 / 2 (61% / 28% / 11%) | full sample clusters on one day |
| EPOCH-2 + both patches (v0.2 extended) | 2026-04-22 ~ 05-15 | TBD | Pending v0.3 sampling protocol | enforcement layer not yet deployed |

**Cohort comparability warning**: rows do not represent a stable cohort; the table is for time-line visualization only and **cannot serve as evidence of pre-vs-post-patch correctness improvement**.

---

## 6. Discussion

### 6.1 The recurring failure assumption

Each epoch's design assumed a capability the deployment data later invalidated:

| Epoch | Assumed capability | Empirical outcome |
|---|---|---|
| EPOCH-0 | AI agents will self-organize like human departments | Coordination overhead dominates (97%) |
| EPOCH-1 | 20 rules suffice for self-governance | Rule-conflict combinatorics break the system |
| EPOCH-2 | LLMs possess meta-cognition to fire axioms | Axioms exist but do not auto-trigger |
| EPOCH-2 + PATCH-001 | Post-action verification suffices | Action proceeds from un-grounded premises |
| EPOCH-2 + Wuji Patch | Structural grounding at both *hua* and *zhen* | Open: under-firing of self-doubt remains |

### 6.2 Implications for LLM-governance frameworks

The strongest claim we extract is: **any framework requiring the LLM to autonomously decide *when* to apply a rule will fail in deployment**. Triggering must be structural — embedded in the loop primitives, in tool-use protocols, or in input-output schemas — not in the LLM's introspective judgment.

This has design consequences:

1. **Boundary conditions ("rules") should be enforcement targets, not decision targets.** They are checked by the framework, not selected by the agent.
2. **Loop primitives should carry mandatory side-effects.** "Verify" must produce a citation; "act" must consume a grounded premise.
3. **Self-doubt cannot be decorative.** A healthy fuzzy ratio (~20%) is a system-level instrumentation requirement, not a desideratum.

### 6.3 The remaining open problem

Even after Wuji Patch, the fuzzy ratio is 11%, below our 20% monitoring anchor (which §5.3 now marks as `[unverified]`). The agent is *over-confidently* applying grounded premises — it grounds correctly when grounding is required, but does not initiate self-doubt for premises that *appear* grounded but are tangentially relevant or out-of-distribution.

This is structurally analogous to the original meta-cognition problem, displaced one level up. **v0.1's prediction**: the next patch will likely target the *gan* (perceive) phase: grounding the *intake* of premises, not just their use.

**v0.2 addendum**: this prediction is one of three parallel routes (§6.4); v0.2 no longer single-bets on it.

### 6.4 Reverse hypothesis (v0.2 new)

v0.1's conclusion implicitly assumed the patch pathway is viable — that adding a *gan*-phase patch would bring fuzzy rate toward the healthy threshold. After 7 days of additional observation and a systematic review, v0.2 makes three parallel routes explicit, each with an abandonment condition:

**Route A — *gan*-phase patch (v0.1's prediction)**: add a grounding clause to the *gan* primitive (e.g., upon premise intake, the agent must distinguish "external source citation" from "inference re-statement").

**Route B — Enforcement-layer reinforcement**: stop adding loop-primitive patches; add enforcement structure (hooks, schemas, external verifiers; see §4.4).

**Route C — Patch pathway fundamentally insufficient**: the loop-primitive ceiling has been approached; the next move requires architectural change (RLHF on metacognition, externally grounded verifier model, multi-provider ensemble).

**Abandonment conditions** (any of which triggers re-evaluation):

1. After adding the *gan*-phase patch, fuzzy rate remains < 15% for 30 days (Route A failure signal)
2. After enforcement layer deployment, drift still observed at hooked positions (Route B failure signal)
3. After ≥ 5 total patches with fuzzy rate < 15% (Route C confirmation: patch pathway ceiling reached)

**v0.2 stance**: under current data, none of the three routes can be ruled out; we recommend parallel exploration rather than the v0.1 single-bet on Route A.

### 6.5 LOCAL feedback loop ≠ Wuji (v0.2 new)

v0.1 focused on whether the wuji prompt-level patch worked. During v0.2's observation window we identified a second grounding mechanism that v0.1 did not acknowledge:

**The 變通 (adaptation) mechanism** (CLAUDE.md §變通): each Owner correction → agent writes `memory/feedback_*.md` → in-context retrieval on similar situations prevents recurrence. This is RLHF-via-correction in character; **it is not the Wuji Patch doing the work**.

This pathway operates only when **external forcing functions** are present: (a) Owner is engaged and corrects, (b) the agent writes the memory. Neither is enforced by the wuji patch; both rely on:
- (a) Owner's active investment
- (b) the affordance provided by `scripts/axiom-log.sh` plus the CLAUDE.md §推導審計 prompt-level reminder "if not logged, it did not happen"

**Acknowledging this does not weaken v0.1's thesis** — the claim that LLM agents lack meta-cognition remains valid. The adaptation mechanism works *because* external forcing functions (Owner + script affordance) fill the gap, *not because* the LLM introspects. v0.2 simply decomposes the observed grounding capability into "Owner intervention (strong) + memory feedback loop (medium) + wuji prompt-level reminder (weak)," to avoid attributing the full result to wuji.

Empirical signal: in the 24 days following v0.1's seal, `memory/feedback_*.md` grew from ~10 entries to 60+ entries, each tied to a specific Owner correction and subsequent in-context recurrence-prevention. This RLHF-via-correction growth curve is **more pronounced than fuzzy-rate improvement**, indicating it carries the bulk of the LOCAL grounding work.

---

## 7. Limitations

- **Single-operator setting**: the framework is deployed for one Owner across multiple projects; multi-operator dynamics are unverified.
- **Sample size**: n=18 is descriptive, not inferential. Logging cadence needs structural improvement.
- **No control condition**: the comparative effect of each patch is observed sequentially, not via random assignment.
- **LLM specific to one provider**: deployment runs on a single model family; cross-provider generalization unverified.
- **Closed corpus**: source artifacts (`archive/epochs/*.md`, `mono/data/axiom-log.csv`) are private. Reproducibility requires public release.

---

## 8. Conclusion

Three epochs and two patches over 24 days (v0.2: extended to 31 days) demonstrate a recurring pattern: each governance design overestimates LLM capability, deployment exposes the gap, and the next iteration moves more enforcement out of the agent's introspective judgment and into structural triggers in the framework loop. The Wuji Patch — grounding the action phase before it executes — places a grounding requirement at the correct position in the loop (*hua*), closing the positional gap left by post-action verification alone.

**v0.2 revised reading**: the remaining problem of under-firing self-doubt (fuzzy rate 11% vs anchor 20%) corresponds to three parallel routes (§6.4) — *gan*-phase patch / enforcement-layer reinforcement / patch pathway fundamentally insufficient — no longer a single bet on v0.1's *gan*-phase prediction. Concurrently, the 7-day extended observation revealed that real grounding capability in LOCAL deployment is **Owner intervention + memory feedback loop (RLHF-via-correction) + wuji prompt-level reminder, three layers stacked** (§6.5), not wuji alone.

We submit this as a small empirical contribution to the question of how to govern LLM agents whose meta-cognition cannot be assumed. **v0.2's added answer**: (a) patches should be placed at the correct positional structure in the loop (v0.1 achieved this), but (b) enforcement structure cannot be omitted (v0.1 left this open), and (c) external forcing functions (Owner, scripts, hooks) must be acknowledged as the actual carriers of grounding; the patch is positional scaffolding, not the engine.

### 8.5 EPOCH-3 forecast (v0.2 new)

Predicted next failure modes (observation points that would trigger EPOCH-3 restructuring):

1. **Fuzzy rate stuck at 11%-15% for ≥ 90 days** (even after a *gan*-phase patch) → patch-pathway ceiling confirmed, architectural change required
2. **Single-provider lock-in exposure**: cross-LLM-family migration (e.g., Anthropic → OpenAI) reveals that patch-text grounding requirements lose semantic effect (different RLHF training distributions) → patch portability becomes a real concern
3. **Memory dilution accelerates**: when feedback memory exceeds 200 entries, the hit-rate for the agent's grounding lookups drops (already observed at the v0.1-to-v0.2 inflection: memory grew from ~10 to 60+ entries) → memory indexing (grep/embedding) becomes necessary or wuji self-undermines
4. **The *gan*-phase patch has a meta-cognition prerequisite**: distinguishing "external source citation" from "inference re-statement" *at intake* itself requires meta-cognition; if so, Route A self-contradicts v0.1's main thesis ("meta-cognition cannot be assumed"), forcing a transition to Route B or C

The core question of EPOCH-3 is unlikely to be "another patch" and more likely "the ratio between patches, enforcement, and architectural change."

---

## Acknowledgments

To the four corrections during the control-flow design incident, which produced PATCH-001. To the question "化的時候，你怎麼知道你的前提是真的？" ("when you act, how do you know your premises are true?") which produced the Wuji Patch.

**v0.2 acknowledgment**: to the 11 argument gaps surfaced during a 2026-05-15 review session — the conflation of positional vs enforcement structure under "structural closure," the grounding gap of the 20% anchor, the implicit single-bet assumption on the patch pathway. This v0.2 revision retains v0.1's core contributions, strengthens its honesty, and does not weaken its thesis. The review tool itself applied wuji's *zhen* primitive to its self-coherence check (Appendix §F) — a case of wuji's design being usable to evaluate wuji itself.

*Acknowledgments use a non-standard academic tone, reflecting the framework's personal origin. Readers may treat as a narrative appendix without consequence to the argument.*

---

## References

1. *I Ching*, "Xici" appendix, "太極→兩儀→四象→八卦" passage. Traditional commentary on cosmogonic compression.
2. shared-brain repository, `archive/epochs/EPOCH-0_department.md`. Internal post-mortem, 2026-04-09.
3. shared-brain repository, `archive/epochs/EPOCH-1_shared-brain.md`. Internal architecture transition note, 2026-03-30.
4. shared-brain repository, `archive/epochs/EPOCH-2_axiom-derivation.md`. Internal framework documentation, 2026-04-22.
5. shared-brain repository, `mono/data/axiom-log.csv`. Decision-correctness logging, 2026-04-13 to 2026-04-22.
6. Kadavath, S. et al. (2022). *Language Models (Mostly) Know What They Know*. arXiv:2207.05221.
7. Tian, K. et al. (2023). *Just Ask for Calibration: Strategies for Eliciting Calibrated Confidence Scores from Language Models Fine-Tuned with Human Feedback*. arXiv:2305.14975.
8. Hart, H.L.A. (1961). *The Concept of Law*. Oxford University Press. (Conceptual reference for rule-vs-principle compression.)
9. shared-brain repository, `archive/epochs/wuji-saga.md`. Narrative companion to this paper, 2026-05-08.
10. Internal control-flow design incident commits (anonymized), 2026-04-14 to 2026-04-15. Triggering case for PATCH-001.

---

## Appendix A: Loop primitive definitions, post-Wuji-Patch

```
感 (gan)  — perceive: receive signals from environment, Owner, or worker outputs
化 (hua)  — act: must be grounded (Wuji Patch). technical premises must point to source code,
            specifications, or measurements; otherwise tag [unverified] and pause to verify
貞 (zhen) — verify: must ground (PATCH-001). compare against external facts, not against
            one's own derivations
新 (xin)  — settle: pattern recognition, memory consolidation, lesson formalization
```

## Appendix B: 爻則 boundary conditions, full text (translated)

1. Emotion is signal, not command.
2. Repetition is redundancy: eliminate or systematize.
3. Most things extreme-conservative, a few extreme-aggressive, never the middle.
4. Bet what you can afford to lose; avoid what would zero out the option space.
5. Unverified does not count.
6. Factual claims must carry sources; no source = unverified = not a decision input.
7. Ask "why" at least three layers before solving a problem.
8. Before acting, find the root cause: But-For + recurrence test + explanation test; failing all three, it is not a root cause.

(With Wuji Patch, the loop primitive *hua* enforces clauses 5, 6, 8 structurally before action begins.)

---

## Appendix C: Self-coherence Audit (v0.1 — 2026-05-08; v0.2 supplement — 2026-05-15)

### C.0 External dependency statement of the audit mechanism (v0.2 new)

The act of performing a self-coherence audit appears, at first glance, to contradict v0.1's main thesis ("LLM agents lack meta-cognition and cannot autonomously verify"). If the agent truly has no meta-cognition, how can §C audits be performed by the agent itself?

Resolution: **§C's audit capability derives from external artifact comparison + external trigger, not from introspection**:
- **External artifacts**: cross-file diffs (against wuji-saga.md, project_epoch2.md, project_wuji_patch.md), grep of existing memory, git log comparison — all of these are the agent **reading** fixed external data, not introspecting.
- **External trigger**: a single-word "自洽?" (self-coherent?) from Owner initiates the audit. **The Owner trigger itself is an instance of a forcing function** — without it, the audit does not autonomously occur (v0.1 to present, §C has only run when Owner prompted).

In other words, §C audits exemplify §6.5's "real grounding capability in LOCAL = Owner intervention + memory feedback loop + wuji prompt-level reminder, three layers stacked." The audit work is mostly carried by the first two layers; the wuji prompt-level reminder is only a triggering posture template.

v0.1 §C omitted this statement, creating a surface contradiction; v0.2 closes it.

### C.1 ~ C.5 (v0.1 audit results, retained)

Following the Wuji Patch's *貞必接地* clause, this paper underwent a self-coherence audit. The audit checked five dimensions; results below.

### C.1 Cross-document data consistency
Numerical claims (n=18, 61% correct, 28% wrong, 11% fuzzy) and timeline anchors (2026-03-30, 2026-04-11, 2026-04-15, 2026-04-22, 24-day evolution) appear identically in this paper and the companion narrative document `wuji-saga.md`. **Pass.**

### C.2 Memory alignment
Internal memory entries `project_epoch2.md` and `project_wuji_patch.md` were checked against this paper's claims. The "agent meta-cognition is an erroneous assumption" thesis, the patch ordering (PATCH-001 before Wuji Patch), and the loop-primitive modification approach all match. **Pass.**

### C.3 Anonymization completeness
A revision pass on 2026-05-08 removed (a) revenue-related descriptions and (b) hardware-specific identifiers from the triggering case for PATCH-001. Post-revision grep for revenue / hardware-name / control-variable-name terms returns zero hits. **Pass.**

### C.4 Industry-positioning fact-checking
The 2024–2026 architectural lineage table (§2.5) initially listed Anthropic Constitutional AI under Stage 4 (spec-driven + memory layer). This was incorrect: Constitutional AI is a 2022 training-time alignment technique, not a 2025–2026 deployment-time spec layer. Corrected on 2026-05-08; replaced with Claude Skills, Cursor Rules, and skills-manager (which do operate at the spec layer). A footnote in §2.5 records this distinction. **Corrected.**

### C.5 Open issues remaining
- **Sample concentration**: n=18 cluster on a single day (04-13) limits statistical power. Logging cadence improvement is required for any inferential claim.
- **Self-doubt under-firing**: Fuzzy ratio of 11% (vs. healthy threshold ~20%) indicates the patches close the structural gap but not the calibration gap. A future patch on the *gan* (perceive) phase is hypothesized but not validated.
- **Single-provider deployment**: All data come from one LLM family. Cross-provider generalization unverified.

*Audit performed by the agent itself, after Owner prompted with the single word "自洽?" (self-coherent?). The act of asking this question is, structurally, an instance of the* 貞 *primitive firing on the paper itself.*

### C.6 v0.2 revision self-coherence audit (2026-05-15)

Applying the same *zhen* primitive to v0.2 revisions:

- **C.6.1 v0.1 ↔ v0.2 compatibility**: all v0.2 additions (§4.4 / revised §5.3 / §5.4 / §5.5 / §6.4 / §6.5 / §8.5 / §C.0 / acknowledgment supplement) **reinforce** rather than **refute** v0.1. v0.1's three core contributions are preserved: (a) failure-mode taxonomy, (b) meta-cognition cannot be assumed, (c) modify-loop-not-rule design principle. v0.2 splits "structural" into two dimensions, treats selection bias, makes three reverse-hypothesis routes explicit, and acknowledges the LOCAL feedback loop — all moves that make v0.1's argument stand more firmly. **Pass**.

- **C.6.2 Affirming-tone consistency**: the "v0.2 Revision Note to v0.1 readers" (top of paper), §8 conclusion ("v0.1 achieved / v0.2 opened"), and acknowledgment ("retains core contributions, strengthens honesty, does not weaken thesis") are mutually consistent. **Pass**.

- **C.6.3 Grounding of new claims**: §5.4's "Owner-correction count method" is explicitly marked as `[pending]` (computation deferred); §6.4's "30-day / 90-day abandonment conditions" are design choices, not empirical findings; §6.5's "memory grew to 60+ entries" is verifiable via `ls projects/memory`. All three are tagged appropriately. **Pass**.

- **C.6.4 Consistency with LOCAL CLAUDE.md**: §6.5's description of the LOCAL feedback loop aligns with the CLAUDE.md §變通 "2026-05-15 self-coherence statement" (both updated the same day). **Pass**.

- **C.6.5 External cross-reference**: v0.2 changes are driven by `archive/epochs/wuji-review-2026-05-15.md` (same-day review document) and its 11 gaps (B1-B11) + 3 LOCAL observations (L1-L3). Mapping:
  - B1 → §4.4 dimensional split + §8 conclusion adjustment
  - B2 → §5.4 selection-bias floor
  - B3 → §5.3 anchor `[unverified]`
  - B4 → §3.2 weakened N² claim
  - B5 → §C.0 external dependency statement
  - B6 → §6.4 reverse hypothesis
  - B8 → §2.5 timeline specifics
  - B9 → §5.5 trend table
  - B11 → acknowledgment "narrative appendix" footnote
  - L1-L3 → §6.5 LOCAL feedback loop
  - B7 / B10 / C1-C4: B7 subsumed in B1; B10 / C1 / C3 deferred to v0.3 (Appendix D + EPOCH-3 forecast — §8.5 partially covers C2); C4 95% CI is limited by n=18, §5.5 provides descriptive trend table instead.

  **Pass — 90% mapped, 10% deferred to v0.3**.
