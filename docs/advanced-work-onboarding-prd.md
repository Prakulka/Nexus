# PRD: Advanced Work onboarding in the Nexus registration flow

| | |
|---|---|
| **Status** | Draft for team review |
| **Author** | Pranita Kulkarni |
| **Source** | BizChat_Advanced_Work_MCP_Onboarding_Design.docx (Larissa Gomes de Stefano Escaliante) |
| **Contributors** | Larissa Gomes de Stefano Escaliante, Advanced Work crew, Inception Bench team, Alchemy vTeam |
| **Last updated** | 2026-08-10 |

---

## 1. Summary

Today a partner can complete the BizChat Built-In MCP onboarding in Nexus and still
miss everything required to ship the same tool for **Advanced Work (FluxV4 / inner-grain)**.
Advanced Work readiness — Code Harness integration, FluxV4 SEVALs, Inception Bench —
lives in separate docs, review forums, and tribal knowledge instead of in the official
Nexus onboarding flow.

This PRD extends the existing Nexus registration flow into a **single onboarding
lifecycle** that covers both:

1. **BizChat mainline / outer-grain** scenarios, and
2. **Advanced Work (FluxV4) / inner-grain** scenarios.

Rather than standing up a parallel Advanced Work process, we reuse the existing
Nexus + Alchemy flow and add explicit Advanced Work readiness gates, evaluations, and
rollout requirements.

**The core requirement for every change below: each Nexus task must state its own
applicability context.** A task carries one of two labels:

- 🟦 **ALL tools** — required for every tool onboarding through Nexus.
- 🟪 **Advanced Work only** — required only when the tool is being onboarded for
  Advanced Work. If a partner is not onboarding for Advanced Work, the task is skipped,
  and the task copy must say so.

> **Why the labels matter (the ask):** if a SEVAL (e.g. FluxV4) is only required for
> Advanced Work, the Nexus task must say "complete this only if this tool is for
> Advanced Work." Likewise the Inception Bench task must note that it needs to be
> completed only if the tool is for Advanced Work. No task should be ambiguous about
> whether it applies to the partner in front of it.

> **Scope note (Pranita, review comment):** although the source doc is framed around
> MCP, the onboarding scope should also cover **Plugins** and **Agent Skills**, not
> MCP only. Task copy should be written tool-type-agnostic where possible.

---

## 2. Current problems

| Problem | Applies to |
|---|---|
| A partner can finish Built-In plugin onboarding and still miss Advanced Work requirements. | 🟪 Advanced Work |
| Inception Bench evaluation exists separately from the standard flight-review evaluation. | 🟪 Advanced Work |
| Teams must complete separate workflows, reviews, and forums for inner-grain onboarding. | 🟪 Advanced Work |
| Operational overhead from maintaining outer-loop and inner-loop onboarding separately. | 🟪 Advanced Work |

---

## 3. Current ("as-is") flow

### 3.1 Outer-grain (BizChat mainline) — the canonical path

| # | Task | Applies to |
|---|---|---|
| 1 | Generate trigger / non-trigger query sets (SubstrateTools **QuerySetGen**). | 🟦 ALL tools |
| 2 | Copilot Design Review — descriptions, trigger queries, conflict assessment, architecture (async, Alchemy vTeam). | 🟦 ALL tools |
| 3 | MCP registration in Nexus — Nexus generates integration artifacts + onboarding PR. | 🟦 ALL tools |
| 4 | Run SEVALs — 1K, DA, RAI regression, Shadow experiment. | 🟦 ALL tools |
| 5 | Flight review — progression SDF → MSIT → WW. | 🟦 ALL tools |
| 6 | Compliance sign-offs — Compliance, Security, Privacy, RAI. | 🟦 ALL tools |

### 3.2 Inner-grain (Advanced Work) — not yet centralized

| # | Requirement | Applies to |
|---|---|---|
| A | Create Helix patch — add plugin module to the Code Harness list. | 🟪 Advanced Work only |
| B | Run Advanced Work evals — FluxV4 SEVALs, existing feature query set, Inception Bench regression set; fill the **Inception Bench Intake Request**. | 🟪 Advanced Work only |
| C | Attend "Inner Loop – CLI Technical Design Review Slots." | 🟪 Advanced Work only |
| D | Get approval from the **AdvancedWork crew** and **BizChat Deepwork BPR** before SDF/MSIT/WW rollout in Code Harness. | 🟪 Advanced Work only |
| E | Attend "Inception Bench Evals – Partner Onboarding Weekly Sync." | 🟪 Advanced Work only |

---

## 4. Proposed unified flow ("to-be")

Order reflects Pranita's review comment: **design review cannot happen before Nexus
intake.** The corrected sequence is intake → trigger queries → design review →
auto-generated code → evaluations → flighting.

### Task 1 — Nexus tool intake
Partner submits the Nexus intake first. This is the entry point for every tool type
(MCP, Plugin, Agent Skill).
- **Applies to:** 🟦 ALL tools.
- **Advanced Work context:** intake form must capture an **"Is this tool for Advanced
  Work?"** answer, because that single answer drives which 🟪 tasks below become required.

### Task 2A — Create trigger queries
Expand from outer-grain only to also cover inner-grain / Advanced Work and
create-task-style Advanced Work flows, producing **one evaluation set covering both
execution modes**.
- **Applies to:** 🟦 ALL tools generate outer-grain queries.
- **Advanced Work context:** 🟪 inner-grain + create-task query coverage is required
  **only if the tool is for Advanced Work**. Existing standard tools stay outer-grain
  only unless they are onboarded into Advanced Work.
- **Open item (Larissa):** FluxV4 uses **ChecklistLeo** while mainline uses **LMC
  Checklist**, and ChecklistLeo is not fully supported by SEVAL — align on the final
  query-set format and SEVAL support before this task is mandatory.

### Task 2B — Copilot Design Review
Existing review scope (descriptions, trigger quality, architecture) plus new scope
(Advanced Work readiness, inner-grain scenarios).
- **Applies to:** 🟦 ALL tools.
- **Advanced Work context:** the Advanced Work readiness review and an **Inception
  Bench reviewer** (suggested: Maryna, Vinay) are added **only when Advanced Work
  support is intended**.

### Task 3 — Nexus registration / auto-generated code
Extend Nexus code generation to emit **mainline integration** and, when applicable,
**Advanced Work Code Harness integration**, potentially behind the same flight config.
- **Applies to:** 🟦 ALL tools get mainline integration generated.
- **Advanced Work context:** 🟪 the additional Code Harness integration is generated
  **only if the tool is for Advanced Work**. (Pranita: proposal is feasible; provide
  **sample PRs** demonstrating the pattern.)

### Task 4 — Unified evaluations
| Evaluation | Applies to |
|---|---|
| 1K SEVAL | 🟦 ALL tools |
| DA SEVAL | 🟦 ALL tools |
| RAI SEVAL | 🟦 ALL tools |
| Shadow experiment | 🟦 ALL tools |
| **Inception Bench** | 🟪 **Advanced Work only** — complete only if the tool is for Advanced Work |
| **FluxV4 validation** | 🟪 **Advanced Work only** |
| **Inner-grain validation** | 🟪 **Advanced Work only** |

- **Advanced Work context:** the task must explicitly tell the partner that Inception
  Bench, FluxV4, and inner-grain validation are skipped for non–Advanced Work tools.

### Task 5 — Unified flight review
One rollout path to SDF → MSIT → WW, without separate Advanced Work review forums.
- **Applies to:** 🟦 ALL tools.
- **Advanced Work context:** 🟪 **Inception Bench regression evidence** is required in
  the flight review **only for Advanced Work–enabled tools** (modeled on the DA
  evaluation evidence requirement).

---

## 5. Nexus gap analysis & action items

Every action carries an applicability label so the resulting Nexus task copy inherits it.

### 5.1 Pre-coding gate

**Query set generation** — guidance covers only standard trigger/non-trigger sets;
missing Advanced Work scenarios and complex inner-grain trigger coverage.

| Action | Owner | Applies to |
|---|---|---|
| Edit Nexus task description to call out Advanced Work coverage | Nexus / Playbook owners | 🟪 Advanced Work gap |
| Update the playbook | Nexus / Playbook owners | 🟪 Advanced Work gap |
| Improve QuerySetGen automation for inner-grain | QuerySetGen team | 🟪 Advanced Work |

**vTeam async review** — Advanced Work requirements are not reviewed today.

| Action | Owner | Applies to |
|---|---|---|
| Align vTeam on Advanced Work requirements | Alchemy vTeam | 🟪 Advanced Work |
| Add an Inception Bench reviewer (e.g. Maryna, Vinay) | Inception Bench team | 🟪 Advanced Work |

### 5.2 Build

**Flight review submission** — Inception Bench is not part of the standard flight-review
job template.

| Action | Applies to |
|---|---|
| Add Inception Bench to the Flight Review Job Group | 🟪 Advanced Work |
| Support ChecklistLeo assertions in SEVAL | 🟪 Advanced Work |
| Define pass/fail thresholds | 🟪 Advanced Work |
| Define regression-handling templates | 🟪 Advanced Work |

Dependencies: ChecklistLeo, LMC assertions, SEVAL platform.

### 5.3 Quality

| Action | Applies to |
|---|---|
| New task: run **Inception Bench regression test** (non-regression check) | 🟪 Advanced Work only |
| Add Advanced Work–specific quality guidance (inner-grain visibility beyond precision / recall / parameter accuracy) | 🟪 Advanced Work |
| Add **Code Harness flighting** instructions (`module_codeHarnessPluginList_deepWorkAndO365Dual.json`, `codeHarnessPluginList.module.json`) | 🟪 Advanced Work |
| Update generated code path for Code Harness flighting | 🟪 Advanced Work |
| Add Advanced Work MCP calls to the DevUI **MCP tab** (container-grain invocation) | 🟪 Advanced Work |
| Add Advanced Work E2E validation guidance | 🟪 Advanced Work |

### 5.4 Test cluster

| Action | Applies to |
|---|---|
| Add **Inception Bench execution** to test-cluster validation | 🟪 Advanced Work |

### 5.5 Ring progression

| Action | Applies to |
|---|---|
| Add **Advanced Mode Template Request Intake** to the WW-enable step | 🟪 Advanced Work |
| Investigate missing Power BI plugin telemetry in Nexus observability | ⬜ General |
| Verify monitoring covers the Advanced Work path | 🟪 Advanced Work |
| Document rollback criteria | ⬜ General |
| Document reliability expectations | ⬜ General |

### 5.6 Other

| Action | Applies to |
|---|---|
| Add a new Nexus task category **Advanced Mode Template Request Intake**, parallel to Security / Privacy / RAI signoff | 🟪 Advanced Work only |

---

## 6. Review feedback captured

- **Pranita Kulkarni** — onboarding scope should not be limited to MCP; include
  **Plugins** and **Agent Skills**.
- **Pranita Kulkarni** — Nexus registration scope should extend beyond MCP registration
  to **Agent Skills**.
- **Pranita Kulkarni** — reorder to: (1) Nexus intake, (2) trigger queries, (3) design
  review, (4) auto-generated code, (5) evals, (6) flighting — design review depends on
  Nexus intake.
- **Pranita Kulkarni** — Code Harness proposal is feasible; provide **sample PRs**.
- **Larissa Gomes de Stefano Escaliante** — FluxV4 (**ChecklistLeo**) vs mainline
  (**LMC Checklist**) mismatch; ChecklistLeo not fully SEVAL-supported. Align on the
  final query-set format and SEVAL support.

---

## 7. Open questions

- **Nexus:** How are Nexus tasks added or modified? How does code generation work today?
  How can Advanced Work requirements be added to code generation?
- **Rollout:** Should Advanced Work reuse existing forums or separate forums? Should all
  new MCP servers also onboard to Advanced Work? One shared trigger-query set, or
  separate inner/outer sets?

---

## 8. Applicability summary

### Required for ALL tools (mainline)
Nexus intake · query generation · design review · MCP registration ·
1K SEVAL · DA SEVAL · RAI SEVAL · Shadow experiment · flight review ·
Compliance / Security / Privacy / RAI signoff.

### Advanced Work–only (complete only if the tool is for Advanced Work)
Code Harness integration · Helix patch · FluxV4 validation · Inception Bench intake ·
Inception Bench regression testing · CLI Technical Design Review · Deepwork BPR approval ·
Advanced Work rollout approvals · Advanced Mode Template Request intake ·
Advanced Work–specific quality evaluation · Advanced Work flighting guidance ·
container-grain debugging guidance · Advanced Work observability validation.
