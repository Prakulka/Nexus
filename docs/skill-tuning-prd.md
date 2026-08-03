# PRD: Skill Tuning in the Nexus Agent Skills Flow

| | |
|---|---|
| **Status** | Draft for team review |
| **Author** | Pranita Kulkarni |
| **Contributors** | Robert Ruara (Skill Tuning Toolkit / SEVAL), Viktoryia Trukhan |
| **Last updated** | 2026-08-03 |
| **Source** | "Skills tuning in Nexus Agent Skills flow" review (2026-08-03, 27 min) |

---

## 1. Summary

Skills registered through Nexus frequently ship with **under-triggering** and **poor discovery** because the skill's `name`, `description`, and front matter are never evaluated before the skill is flighted. Today, tuning happens late — after the PR is generated and Sydney DRIs are already in the loop — which is slow and expensive.

This PRD proposes embedding the **Skill Tuning Toolkit** (`plugin-eval` → `tune-skill.py` → `sydneyflow` → SEVAL) directly into the Nexus skill registration & validation flow so that authors catch and fix quality problems **at authoring time**. We introduce two capabilities:

1. **Front-matter tuning (upfront, in-flow):** a one-click static-analysis + auto-optimization pass on the skill file (token budget, YAML formatting, scorecard).
2. **An Evaluate phase (deeper):** query-set–driven trigger-quality tuning that runs a live inner-loop trigger eval against M365 Copilot, with SEVAL as the at-scale outer loop.

## 2. Background & context

### 2.1 Current Nexus skill flow

```
Registration (name, description, ownership, ICM, business justification)
        │
        ▼
Alchemy triage  ─►  PM triage  (async, weekly; mostly approve; redirect non-skills)
        │
        ▼
Registration & Validation
   1. Define skill
   2. Skill definition (upload / paste SKILL.md)
   3. Query set definition
   4. Upload scripts & references
   5. Configure flight  ─►  flight config  ─►  flighting in CCS
        │
        ▼
PR generated  ─►  Sydney DRI review & approve
        │
        ▼
Test & Debug (pre-merge / post-merge)  ─►  SEVAL
```

### 2.2 Why the model under-triggers

For a skill, the **only** thing the routing model sees is the **`name` and `description` from `SKILL.md`** (not the high-level intake description entered on the registration page). If the description doesn't clearly prescribe *when* the skill should be used, the model won't invoke it. Therefore tuning must operate on the **final SKILL.md content**, and query generation must be derived from that same content — not from the intake blurb.

> Observed example of a too-thin intake description: *"Skill for content control document operations. Use this skill when user asks to apply content controls formatting or operation."* — not enough signal to generate a good query set or to trigger reliably.

### 2.3 The Skill Tuning Toolkit (Robert's team)

A repeatable, script-driven pipeline:

```
Static Analysis  →  LLM Optimization  →  Live Trigger Eval  →  At-Scale Regression
 (plugin-eval)      (tune-skill.py)       (sydneyflow)          (SEVAL)
```

- **`plugin-eval`** — static analysis (built on OpenAI's plugin-eval engine). Produces a scorecard + token budget breakdown (trigger / invoke / deferred tokens) and "Fix First" recommendations.
- **`tune-skill.py`** — automated LLM optimization loop; rewrites `SKILL.md` based on `plugin-eval` findings (no manual editing).
- **`sydneyflow` (inner loop)** — live trigger-rate measurement against M365 Copilot by calling **Sydney via deepeval** (~10–15 min for ~30 queries). Preferred over Copilot Playground, which holds tool results constant and can't execute the full flow.
- **SEVAL (outer loop)** — at-scale validation: `lmchecklist` (triggering) + `agenticleosbs` (response quality). Pipeline is **ready** for skills today (SDF flights already present; migrating to MSIT).
- Supporting: `make-config.py` (generates sydneyflow configs), `flights.json` (single source of truth for flight config).

**Proof point:** on a deliberately poor 348-line / 5,457-token `blog-post` skill, one automated iteration moved the score **67 → 95 (D → A)**, cut tokens **89% (5,457 → 586)**, cut lines **86% (348 → 47)**, and validated skills reached an **80%+ inner-loop trigger rate**.

## 3. Goals & non-goals

### Goals
- **G1.** Catch front-matter / token / formatting problems at the **skill-definition step**, before submission.
- **G2.** Ensure the SKILL.md `name` + `description` are tuned for the intended queries so the model triggers reliably.
- **G3.** Introduce a lightweight **Evaluate phase** with a small query set (10–15 queries) and a fast inner-loop trigger eval.
- **G4.** Reduce the amount of late-stage tuning required during Sydney DRI review / SEVAL.
- **G5.** Reuse Robert's toolkit (`plugin-eval`, `tune-skill.py`, `sydneyflow`, SEVAL) rather than building new eval logic.

### Non-goals (this iteration)
- **NG1.** Tuning of uploaded **scripts** (deferred — only enforce script/skill **name consistency**).
- **NG2.** Replacing SEVAL — SEVAL remains the at-scale outer loop.
- **NG3.** Final decision on **Copilot Playground vs. DeepEval Sydney flow** as the eval substrate (tracked as an open question; Robert to align with Thomas & Pooja).
- **NG4.** 3P Skill Builder and Mainline BizChat integrations (future integration points, see §8).

## 4. Proposed experience

### 4.1 Feature A — Front-matter tuning (upfront)

**Where:** Step 2 "Skill definition," next to the SKILL.md upload/paste control, and again as a gate before **Review & submit**.

**Trigger:** author clicks **"Tune front matter."**

**What it does (via `plugin-eval` + `tune-skill.py`):**
- Validates YAML front-matter **formatting** and required fields.
- Computes **token budget** with a breakdown: **trigger / invoke / deferred** tokens, against budget.
- Produces a **scorecard** (score 0–100 + letter grade) and a ranked **"Fix First"** list.
- Offers **one-click auto-optimize** that rewrites SKILL.md and shows a before/after diff (token count, line count, score) for the author to accept or reject.

**Why upfront:** fast, deterministic, no query set required — the cheapest quality win and the biggest immediate token savings.

### 4.2 Feature B — Evaluate phase (trigger quality)

**Where:** a new **"Evaluate"** tab/step after Skill definition (before Configure flight). Conceptually modeled on Copilot Studio / Agent Builder "evaluate" experiences.

**Step 1 — Generate query set (10–15 queries):**
- Auto-generate from the **SKILL.md `name` + `description`** (the final content the model sees), not the intake description.
- Include **positive** (should-trigger) and **negative** (should-not-trigger) queries so we measure both trigger rate and over-triggering.
- Author can edit / add / import queries.

**Step 2 — Define expected results / checks:** *(open design — see §7)*
- For plugins we assert on function calls + params; skills have no equivalent function-call signal.
- Need to decide the check shape: trigger yes/no, expected script/action, or LLM-judged response. Tooling selection precedes query design.

**Step 3 — Run inner-loop trigger eval (`sydneyflow` → Sydney via deepeval):**
- Runs live against M365 Copilot, ~10–15 min for ~30 queries.
- Report: **overall trigger rate**, per-query pass/fail, over-trigger flags.

**Step 4 — Iterate:** if trigger rate is low, re-tune the `description` (Feature A auto-optimize), regenerate/refresh queries, re-run. Target inner-loop trigger rate **≥ 80%**.

### 4.3 Pre-merge triggering-quality gate + SEVAL

- After PR generation, surface a **pre-merge triggering-quality check** in the Test & Debug panel (wired with Copilot Playground / Dev UI) — likely home for the gate.
- **Post-merge** continues to route to WI for debugging; **SEVAL** (lmchecklist + agenticleosbs) runs as the at-scale regression outer loop.
- Enforce **script/skill name consistency**: if the skill name changes, flag/patch script names so they still resolve.

## 5. Flow, after changes

```
Registration ─► Alchemy/PM triage
        │
        ▼
Registration & Validation
   Define skill
   Skill definition ────────────►  [A] Tune front matter   (plugin-eval + tune-skill.py)
   Evaluate  ★NEW ──────────────►  [B] Generate query set → inner-loop trigger eval (sydneyflow/deepeval)
   Upload scripts & references  (enforce name consistency)
   Configure flight ─► flight config ─► CCS
        │
        ▼
PR generated ─► Sydney DRI review
        │
        ▼
Test & Debug  ─►  [C] Pre-merge trigger-quality gate  ─►  SEVAL (lmchecklist + agenticleosbs)
```

## 6. Success metrics

- **M1.** Median SKILL.md **token count at submission** ↓ (target: within budget; toolkit precedent ~89% reduction on poor skills).
- **M2.** **Inner-loop trigger rate** at submission ≥ **80%**.
- **M3.** Share of skills passing SEVAL triggering on **first** post-merge run ↑.
- **M4.** **Reduction in Sydney-DRI-stage tuning iterations** per skill.
- **M5.** % of registered skills that run front-matter tuning and the Evaluate phase (adoption).

## 7. Open questions

- **Q1. Skill check semantics.** What is an "expected result" for a skill query when there is no function-call assertion (as in plugins)? Trigger-only, expected script/action, or LLM-judged response? *(Owner: Viktoryia + Robert — decide tools first, then query design.)*
- **Q2. Eval substrate.** Copilot Playground vs. DeepEval Sydney flow for the inner loop — Playground may not be representative. *(Owner: Robert w/ Thomas & Pooja.)*
- **Q3. Query generation source.** Confirm generation is driven by SKILL.md `name`+`description`; sample real skills in the MSIT codebase to see what authors actually put in the skill file vs. intake. *(Owner: Pranita.)*
- **Q4. SEVAL docs for skill owners.** Guide + a duplicable reference SEVAL flight. *(Owner: Robert w/ Gaurish.)*
- **Q5. Pre-merge gate mechanics.** Exactly how the pre-merge check is wired into Test & Debug / Dev UI. *(Owner: Nexus team.)*

## 8. Future integration points (beyond this iteration)

The same toolkit plugs into four surfaces; this PRD covers **#2**:

1. **deep-eval (standalone)** — team tooling for evaluating/tuning skills during development (ready today via runbook + scripts).
2. **Nexus (1P onboarding)** — *this PRD*: auto-run `plugin-eval` analyze → scorecard → gate/pass on submission.
3. **Skill Builder (3P customers)** — embed real-time static analysis + "Fix First" + token budget in the UI before submit.
4. **Mainline BizChat (skill creator)** — an `improve-skill` workflow in Copilot at creation time or on demand.

## 9. Deliverables & next steps

| # | Action | Owner |
|---|--------|-------|
| 1 | Figma mockups for Front-matter tuning + Evaluate phase; circulate for feedback | Pranita |
| 2 | Share SEVAL-for-skills doc + duplicable reference flight | Robert (w/ Gaurish) |
| 3 | Recommend which toolkit tool maps to front-matter vs. trigger-quality checks | Robert |
| 4 | Team plays with the Skill Tuning Toolkit to get familiar (runbook + scripts) | Pranita, Viktoryia |
| 5 | Decide skill check semantics (Q1) and eval substrate (Q2) | Viktoryia, Robert |
| 6 | Sample MSIT skills to characterize real SKILL.md name/description quality (Q3) | Pranita |
| 7 | Create work items for the above | Pranita |

## Appendix A — Toolkit package reference

```
skill-tuning-toolkit/
  skill-evaluation-and-tuning-v2.md   # 7-step runbook: evaluate → budget → improve →
                                       #   package → inner loop → outer loop → report
  tune-skill.py                        # automated optimization (plugin-eval + Substrate LLM loop)
  make-config.py                       # generates sydneyflow configs from externalized flights
  flights.json                         # single source of truth for eval flight configuration
```
