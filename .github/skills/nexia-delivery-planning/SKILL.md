---
name: nexia-delivery-planning
description: 'Phase 3.5 (optional) implementation planning and effort estimation. Use when: producing a business-owner-facing delivery plan, comparing alternative implementation strategies, estimating effort and calendar time under different team sizes with and without agentic AI tooling, building a work breakdown with dependencies, identifying parallelizable work streams, or refreshing the plan after scope/tech-stack change. Runs a constraints-first, plans-second two-gate flow. Never blocking.'
argument-hint: 'Project name (Phase 1–3 artifacts are read automatically from ai-driven-development/docs/)'
version: 1.1.0
last_reviewed: 2026-07-20
status: Active
---

# Delivery Planning & Effort Estimation

## Role
**Senior Software Architect & Delivery Manager** — Turn the approved target architecture into a business-owner-facing implementation plan: alternative delivery strategies, transparent effort and calendar-time estimates under multiple team-size scenarios (with and without agentic AI development tooling), a full work breakdown with dependencies, and an explicit map of what can run in parallel. Estimates are decision-support, never commitments.

This phase produces **plans and numbers only** — it writes no application code and changes no target-architecture decision. When a business constraint contradicts the approved target state, it surfaces the conflict rather than silently planning around it.

## When to Use
- **Phase 3.5** — any time after Phase 3 (`target-architecture`) DoD passes and before or during Phase 4. The orchestrator **offers** this phase; the business owner may decline. It is **never blocking**.
- **Standalone / re-run mode** — re-run to refresh the plan when scope, team size, tech stack, or constraints change. On a re-run, diff against the previous plan version and summarize what changed and why (see Step 9).
- Whenever a stakeholder asks "how long", "how many people", "how much", "what's the fastest path", "can AI tooling shorten this", or "what can we do in parallel".

## Prerequisites (Preflight)

| Artifact | Path | Required? |
|---|---|---|
| Legacy analysis | `ai-driven-development/docs/legacy_analysis/legacy_analysis.md` | Always |
| Legacy architecture | `ai-driven-development/docs/legacy_architecture/legacy_architecture.md` | Always |
| Tech stack selections | `ai-driven-development/docs/tech_stack_selections.md` | Always |
| Target architecture | `ai-driven-development/docs/target_architecture/target_architecture.md` | Always |
| UI/UX design artifacts | `ai-driven-development/docs/ui_design/` | Optional (improves page-inventory fidelity) |

**Preflight gate:** Verify each **Always** artifact exists and is non-empty before doing anything else. If any is missing, **list exactly what is missing and stop** — do not estimate against invented inputs.

`ui_design/` is the **only** optional input. When it is absent, derive the page inventory from legacy-analysis screens/flows + target-architecture API design, and mark every derived page row **"derived, pending UI/UX confirmation"**.

**Output directory:** `ai-driven-development/docs/implementation_planning/`
**Standards / templates:** [`STANDARDS.md`](./STANDARDS.md) — HTML+Mermaid page template, gantt/DAG/matrix diagram rules, mermaid validation checklist, and all report table templates.
**Extended artifacts:** [`EXTENDED_ARTIFACTS.md`](./EXTENDED_ARTIFACTS.md) — optional/conditional techniques (AI-gain sensitivity band, portion-split estimation, block defensibility register, open-item crosswalk, customer-facing deck, commercial-language sanitization). Read the relevant section only when its trigger condition is met (see § Step 9.1 and § Estimation Model below).

> **Evidence-based rule:** every inventory item (page, endpoint, service, module, cross-cutting item) MUST trace to a Phase 1–3 artifact (or `ui_design/` when present). No invented pages or services. Cite the source artifact + section for each item.

---

## Two-Gate Planning Flow

Estimation does **not** begin until both gates pass. The strategy space is not the architect's to assume: business owners routinely impose constraints that eliminate whole plan families (for example "no OpenShift — deploy to our VPC only", "keep the existing database as-is and modernize only the application tier", or "no parallel-run, because we cannot fund double infrastructure"). For that reason the flow is strictly ordered — establish the constraints first, agree on the candidate plans second, and only then produce the numbers.

```
Preflight → Gate 1 (Constraints) → Conflict Check → Gate 2 (Plan Set) → Estimation → Report
```

---

### Gate 1 — Expectations & Constraints Elicitation

Run the questionnaire below. Record every answer in the **Constraints & Expectations Register** (a section of `implementation_plan.md`). Classify each item as:

- **Hard constraint** — plans that violate it are excluded from the candidate set.
- **Preference** — scored (not excluded) in the Gate 2 decision matrix.
- **Open** — architect's discretion; record the assumed default.

Every question offers a `default`; if the user answers `default` or skips, record the default and its classification.

**Register schema** (one row per answer):

| ID | Question | Answer | Class (Hard/Pref/Open) | Source of default | Notes |
|---|---|---|---|---|---|

#### A. Strategic / business expectations
1. **Primary driver(s)** — rank if multiple: fastest total delivery · earliest first business value · lowest risk · lowest cost · minimal business disruption · compliance deadline. *(default: earliest first business value, then lowest risk.)* **The ranking here sets the decision-matrix weights in the recommendation.**
2. **Deadline / budget ceiling** — any fixed date or budget envelope the plan must fit. *(default: none — plans compared freely.)*
3. **Cutover tolerance** — acceptable downtime window; is big-bang cutover acceptable (yes/no); appetite for running legacy + new in parallel and paying double run-cost (yes/no). *(default: short maintenance-window cutover; parallel-run acceptable for critical paths.)*
4. **Rollback expectation** — must every milestone be reversible, or is forward-only acceptable after a defined point? *(default: reversible until a named point-of-no-return.)*

#### B. Target-state constraints *(may override or narrow the Phase 3 target architecture)*
5. **Deployment target** — confirm or constrain: cloud provider / VPC / on-prem / Kubernetes / OpenShift / VMs / serverless. **Read the current choice from `tech_stack_selections.md` first** and ask the business owner to confirm it still stands for delivery. *(default: the `tech_stack_selections.md` deployment choice.)*
6. **Database strategy** — keep existing DB as-is (app-tier-only modernization) / migrate engine / migrate schema / both. *(default: whatever Phase 3 designed.)*
7. **Scope reductions / phase exclusions** — anything in the target architecture explicitly out of this plan (e.g. "mobile later", "skip observability stack for v1"). *(default: none — full Phase 3 scope.)*
8. **No-go technologies or vendors** — anything forbidden by policy, licensing, or preference. *(default: none.)*
9. **Coexistence constraints** — must legacy + new share DB, auth, or integrations during transition? Any integration partners that cannot change endpoints? *(default: no forced coexistence; partners can adopt new endpoints on a schedule.)*

#### C. Delivery-model inputs
10. **Team-size scenarios** to model — any set and role mix (backend/frontend/mobile/QA). *(default: 2, 4, and 6 developers.)*
11. **Working-calendar assumptions.** *(default: 5-day week, 6 productive hours/day, no holidays modeled.)*
12. **Seniority mix.** *(default: mixed — 1 senior per 2–3 mid-level.)*
13. **Agentic AI tooling scenario definitions.** *(default: "without" = conventional IDE + autocomplete; "with" = agentic AI coding tools such as Claude Code for scaffolding, implementation, tests, and review support.)*
14. **Scope confirmation** — read the confirmed Phase 4 scope (from Phase 1 Section 10 + orchestrator scope selection) and confirm which sub-phases are in plan scope. *(default: the confirmed Phase 4 scope.)*
15. **Contingency policy.** *(default: PERT three-point estimates + 15% integration-risk buffer on the critical path.)*

#### Conflict detection (mandatory — runs after Gate 1, before Gate 2)
Cross-check every **Hard constraint** against `target_architecture.md` and `tech_stack_selections.md`. If a constraint contradicts the approved target state (e.g. "keep DB as-is" but Phase 3 designed a new schema; "VPC only" but the architecture assumes managed cloud services), do **NOT** silently plan around it. Present the conflict with options:

- **(a)** treat it as a **plan variant** ("Plan X delivers the target architecture minus DB migration"), or
- **(b)** flag that **Phase 3 / Phase 2.5 should be re-run** to revise the target state, or
- **(c)** **withdraw** the constraint.

Record every conflict and its chosen resolution in the register (add a `Conflict / Resolution` note to the affected row). Do not proceed to Gate 2 with an unresolved conflict.

---

### Gate 2 — Plan Set Proposal & Approval

From the constraint register, derive **2–4 candidate implementation plans**. Use the canonical strategy catalog below as the **generation palette** (not a fixed output). Only propose plans compatible with the Hard constraints, and adapt each plan's description to the constraints (e.g. under "keep DB as-is", the strangler plan has no data-migration stream and says so explicitly).

Present the candidate set with a one-paragraph rationale each — **including which constraint eliminated any excluded canonical strategy** — and ask the user to **approve, modify, add, or remove plans before any estimation begins**. The user may also name their own plan; incorporate it as specified. Record the approved set before estimating.

**Canonical strategy catalog (palette):**
- **Big-bang rewrite** — build the complete target system, single cutover.
- **Strangler fig (incremental)** — route-by-route / module-by-module replacement behind a facade, phased cutovers.
- **Hybrid / parallel-run** — backend-first (or frontend-first, per analysis) with full parallel-run verification before cutover.
- **App-tier-only modernization** — new application stack on the unchanged legacy database (the "keep DB as-is" case).
- **Lift-and-reshape** — minimal re-architecture, replatform to the constrained deployment target first, modernize incrementally after.

**Gate 2 is complete only when the user has explicitly approved (or modified) the candidate set.** Estimation starts here.

---

## Procedure

> Do NOT skip, reorder, or summarize steps. Steps 0–2 are the two gates above; Steps 3–9 produce the estimated report.

### Step 0 — Preflight
Verify the always-required artifacts listed in the table above. If any of them is missing, list exactly what is missing and stop before doing anything else. Once the required inputs are present, read all of them: the legacy analysis (screens, flows, integrations, and the Section 10 technology profile), the legacy architecture, `tech_stack_selections.md` (stack and deployment target), the target architecture (services, bounded contexts, API design, and data architecture), and the `ui_design/` artifacts if they are present.

### Step 1 — Gate 1 (Constraints)
Run the questionnaire, build the Constraints & Expectations Register, run the mandatory conflict check, resolve every conflict. Capture the ranked drivers from A1 — they become the decision-matrix weights.

### Step 2 — Gate 2 (Plan Set)
Derive and present 2–4 candidate plans from the palette filtered by Hard constraints; get explicit approval. Record which excluded strategies were eliminated and by which constraint ID.

### Step 3 — Work Breakdown Structure (WBS)
Build the WBS from evidence. Three inventories + cross-cutting items (see **§ WBS Requirements** below). Every item cites its source artifact + section. Assign a complexity rating (5-level scale, with cited criteria — see **§ Complexity Model**).

### Step 4 — Estimation
Apply the estimation model (see **§ Estimation Model**) to every WBS item: three-point estimates (O/M/L), PERT expected value, per-category AI acceleration factor for the "with AI" figure. Keep the math visible and auditable.

### Step 5 — Team-size scaling
For each approved plan × each team-size scenario × {with AI, without AI}, compute calendar duration using critical-path + communication-overhead + resource-leveling (see **§ Team-Size Scaling**). Never divide effort linearly by headcount. Identify the point of diminishing returns per plan.

### Step 6 — Dependency & parallelism analysis
Build the WBS dependency DAG, identify the critical path per plan, group items into parallel swimlanes, and mark the contract-first (OpenAPI stub/mock) enabler as its own WBS item (see **§ Dependency & Parallelism**).

### Step 7 — Diagrams (HTML + Mermaid)
Generate all HTML diagram files per [`STANDARDS.md`](./STANDARDS.md): `dependency_graph.html` (DAG with critical path highlighted), `gantt_<plan>_<scenario>.html` (at minimum the recommended plan, with-AI and without-AI), `scenario_matrix.html` (visual comparison). Then run **Step 7.1 validation**.

### Step 7.1 — Validate generated HTML
Run the **File Creation Validation Checklist** in [`STANDARDS.md`](./STANDARDS.md) for every HTML file, then run `node scripts/validate-mermaid.js` from the repo root (or `--dir ai-driven-development/docs/implementation_planning`). If errors: run `node scripts/validate-mermaid.js --fix`, then correct anything remaining and regenerate the whole file (never patch single lines). A plan with broken diagram renders does not satisfy DoD.

### Step 8 — Recommendation
Compare all approved plans with a scored decision matrix whose criteria weights come from the Gate 1 A1 ranking (time-to-first-value, total duration, risk, cost proxy, business disruption). Recommend one plan + team size + AI scenario, with a per-plan confidence level and the visible planning-estimate banner.

### Step 9 — Assemble outputs & (re-run) diff
Write all files under `ai-driven-development/docs/implementation_planning/` (see **§ Output Artifacts**). If a previous `implementation_plan.md` exists, prepend a **"What changed since the last plan"** section: added/removed plans, changed constraints, changed estimates, and why. Update `redesign_progress.md` (mark Phase 3.5 status). Verify the Definition of Done.

### Step 9.1 — Optional customer-facing artifacts *(conditional — check every trigger, produce only what triggers)*
Do not produce these by default. Check each condition; when true, follow the linked section of [`EXTENDED_ARTIFACTS.md`](./EXTENDED_ARTIFACTS.md) in full.

| Artifact | Trigger | Reference |
|---|---|---|
| `block_defensibility_register.md` | Business owner is billed/planned by effort **and** the recommended plan's gantt has ≥1 block ≥5 work-weeks (≈25 wd) | [§ 3](./EXTENDED_ARTIFACTS.md#3-block-defensibility-register) |
| `open_items_crosswalk.md` | Carried-forward open items exist **and** the business owner (or a review) needs each item's schedule impact and owner action made explicit, beyond the Assumptions Register | [§ 4](./EXTENDED_ARTIFACTS.md#4-open-item-crosswalk) |
| `director_briefing/` deck | A director/executive-level visual readout is needed, distinct from the markdown reports and diagrams | [§ 5](./EXTENDED_ARTIFACTS.md#5-customer-facing-presentation-deck) |

If the deck (or any other artifact a customer will see) is produced, apply commercial-language sanitization to its customer-facing tier before calling it done — [§ 6](./EXTENDED_ARTIFACTS.md#6-commercial-language-sanitization).

---

## WBS Requirements

### 1. Frontend page inventory → `frontend_page_inventory.md`
One row per page/screen from the target design:

| Page | User journey | Complexity (rating) | APIs consumed (service · endpoint · method) | Reused DS components | Effort no-AI (PERT d) | Effort with-AI (PERT d) | AI factor | Depends on (backend endpoints) | Source |
|---|---|---|---|---|---|---|---|---|---|

- **APIs consumed:** at minimum the count; **preferably** the exact `service + endpoint + HTTP method` from the target architecture API design.
- When `ui_design/` is absent, derive pages from legacy screens/flows + target APIs and mark the row **"derived, pending UI/UX confirmation"**.

### 2. Backend service/module inventory → `service_module_inventory.md`
One row per service / module / bounded context from the target architecture:

| Service/Module | Responsibilities | Endpoint count | Integration points | Data-migration involvement | Complexity (rating + rationale) | Effort no-AI (PERT d) | Effort with-AI (PERT d) | AI factor | Depends on | Source |
|---|---|---|---|---|---|---|---|---|---|---|

### 3. Cross-cutting work items *(explicit rows — frequently forgotten in naive estimates)*
Project setup/scaffolding · CI/CD · infrastructure/environments · authn/authz · observability · data migration · security review · test automation (unit/integration/E2E) · UAT support · documentation · cutover rehearsal · cutover. Each is a real WBS row with its own estimate and dependencies.

### 4. Complexity Model — 5-level scale (every rating cites its criteria)

| Level | Label | Criteria signals |
|---|---|---|
| 1 | Trivial | Single entity CRUD, no integrations, no migration, well-known pattern. |
| 2 | Low | A few business rules, 0–1 integration, simple data model, no novelty. |
| 3 | Medium | Moderate business-rule density, 1–2 integrations, moderate data model, some migration. |
| 4 | High | Dense business rules, multiple integrations, complex data model, migration involved, notable NFR constraints. |
| 5 | Very High | Heavy rules + many integrations + complex/ambiguous data + heavy migration + novelty/unknowns + strict NFRs. |

Rate each item on six signals — **business-rule density, integration count, data-model complexity, migration involvement, novelty/unknowns, and non-functional constraints** — and cite which of those signals drove the rating. Never record a rating as a bare number without its supporting rationale.

---

## Estimation Model *(transparent & auditable — all formulas reproducible)*

### 1. Three-point + PERT
Per WBS item, estimate **O** (optimistic), **M** (most likely), **P** (pessimistic) in person-days.

```
PERT expected  E   = (O + 4M + P) / 6
Std deviation  SD  = (P − O) / 6
```

Show O/M/L and E for every item.

### 2. Agentic AI productivity factors *(per-category table, not one global multiplier)*
These are **planning assumptions** — label them as such. If the user supplies historical velocity/team data, prefer it over these defaults. Every "with AI" figure must show **which factor was applied**.

| Work category | Effort reduction (default range) | Planning midpoint |
|---|---|---|
| CRUD / boilerplate / scaffolding | 50–70% | 60% |
| Standard UI pages from a design system | 40–60% | 50% |
| Complex business logic | 20–35% | 27% |
| Integration / debugging with external systems | 15–25% | 20% |
| Data-migration scripts | 30–50% | 40% |
| Test authoring | 40–60% | 50% |
| Architecture / design / review / UAT / coordination | 0–15% | 8% |

```
Effort_withAI = Effort_noAI × (1 − factor)      // factor = category reduction (planning midpoint unless user overrides)
```

The AI factor applies to the PERT expected value of the item's *implementation* effort; it does NOT reduce coordination/review/UAT beyond the last row's range.

**AI-gain sensitivity band (always report, not optional).** Once the per-category factors are applied across the full WBS, compute the PERT-weighted blended reduction — this is the **stretch** figure. Cross-check it against published AI-coding productivity evidence (task-level studies, field trials, RCTs on mature codebases; cite the source class, never fabricate a citation). If the bottom-up stretch exceeds the field-trial range, set a **base** case near the field-trial midpoint and report both — base as the planning/commitment figure, stretch as the labeled "optimistic ceiling, not the expected case." Name a binding **re-measure gate** (the WBS point at which the assumed factor is replaced with observed velocity) as a numbered assumption and a risk-register row. Full method and template: [`EXTENDED_ARTIFACTS.md` § 1](./EXTENDED_ARTIFACTS.md#1-ai-gain-sensitivity-band).

**Portion-split estimation (use when a WBS row is internally heterogeneous).** If a single row mixes sub-work with materially different novelty/integration/business-rule density such that one flat category factor misrepresents it, split the row's O/M/P into cited sub-portions, each with its own category and factor, summing back to the original three-point estimate. A split that changes the row's PERT E is a re-baseline, not a split — label it as such. Method and guardrails: [`EXTENDED_ARTIFACTS.md` § 2](./EXTENDED_ARTIFACTS.md#2-portion-split-estimation).

### 3. Team-Size Scaling *(never linear division by headcount)*
Apply three effects:

**(a) Critical path floor.** Calendar time can never drop below the critical-path length regardless of headcount (see § Dependency & Parallelism).

**(b) Communication overhead (Brooks's Law).** Effective capacity grows sub-linearly with team size `n`. Use the documented pair-channel model:

```
channels(n)      = n × (n − 1) / 2
overhead(n)      = min(0.60, k × channels(n))        // k = 0.015 default; cap 60%
effectiveDevs(n) = n × (1 − overhead(n))
```

Example: n=2 → overhead 1.5%, eff 1.97; n=4 → 9%, eff 3.64; n=6 → 22.5%, eff 4.65; n=8 → 42%, eff 4.64 (diminishing). Record `k` in the Assumptions Register; the user may override it.

**(c) Resource leveling per role.** Split effort by role (backend/frontend/mobile/QA/DevOps). A role's calendar time is bounded by *its own* queue — a frontend dev cannot absorb backend critical-path work. Compute per-role duration, then plan duration = max across roles subject to the critical-path floor.

```
role_calendar_days = role_person_days / (role_effectiveDevs × hours_per_day-normalized)
plan_calendar_days = max( critical_path_days , max_over_roles(role_calendar_days) )
calendar_weeks     = plan_calendar_days / working_days_per_week
```

Report per scenario: **total person-days · calendar duration (working weeks) · average parallel utilization · point of diminishing returns** (the team size beyond which calendar weeks barely improve — where `Δweeks/Δheadcount` flattens).

### 4. Scenario Matrix *(report centerpiece)* → `scenario_matrix.html` + table in `implementation_plan.md`
Rows = Plan A/B/C. Columns = each team size × {with AI, without AI}. Cell = **calendar duration + total person-days**. Include the mermaid/chart visual comparison (see STANDARDS).

### 5. Assumptions Register *(one numbered table)*
Every assumption — productivity factors, `k`, availability, hours/day, scope exclusions, unknowns, contingency % — as a numbered row so a business owner can challenge each individually.

| # | Assumption | Value | Source (default / user-supplied) | Impact if wrong |
|---|---|---|---|---|

---

## Dependency & Parallelism

1. **Dependency graph** — a DAG of WBS items (`dependency_graph.html`, mermaid `flowchart`). Mark **hard technical dependencies** (endpoint must exist before page integration) distinctly from **soft/ordering preferences** (use a dashed edge / edge label per STANDARDS).
2. **Critical path** — identify and highlight per plan; state its length in person-days and which items on it benefit **least** from added people.
3. **Parallel work streams** — group WBS items into swimlanes (Backend Core, Backend Integrations, Frontend, Mobile, Data Migration, DevOps) showing what runs concurrently per team-size scenario. Note **contract-first API design (OpenAPI stubs/mocks)** as the enabler that lets frontend start before backend completes, and include that stub work as its own WBS item.
4. **Gantt charts** — one mermaid `gantt` per (plan × recommended team size), for both with-AI and without-AI variants (`gantt_<plan>_<scenario>.html`). Use **relative weeks** (Week 1, Week 2 …) unless the user supplied a start date.

---

## Output Artifacts

All under `ai-driven-development/docs/implementation_planning/`:

| File | Content |
|---|---|
| `implementation_plan.md` | Master report: constraints & expectations register (with Gate 1 resolutions), approved plan set, WBS, estimation model, scenario matrix, dependency/parallelism analysis, risks, assumptions register, recommendation. On re-run, leads with "What changed". |
| `executive_summary.md` | ≤2 pages, business-owner language, no jargon: scenario matrix, recommended plan + team size, key risks, what "with agentic AI tooling" changes and why. |
| `frontend_page_inventory.md` | Full page × API-consumption × effort table. |
| `service_module_inventory.md` | Full service/module × complexity × effort table. |
| `dependency_graph.html` | Mermaid DAG with critical path highlighted. |
| `gantt_<plan>_<scenario>.html` | Mermaid gantt charts (≥ recommended plan, with-AI and without-AI). |
| `scenario_matrix.html` | Visual comparison of all plan × team × AI scenarios. |
| `block_defensibility_register.md` *(optional, conditional)* | Per-block DEFEND/TRIM/DECOMPOSE verdicts for large gantt blocks. See § Step 9.1. |
| `open_items_crosswalk.md` *(optional, conditional)* | Itemized carried-forward open items × schedule impact × owner action. See § Step 9.1. |
| `director_briefing/` *(optional, conditional)* | Director/executive presentation deck (HTML + PDF), presenter and sanitized customer tiers. See § Step 9.1. |

Every report carries a visible banner: **"⚠ Planning estimate — not a commitment. Confidence: ±X%."** State a confidence level per plan.

---

## Guardrails
- **Never present estimates as commitments.** Visible planning-estimate banner + per-plan confidence, always.
- **No fabricated velocity data.** Productivity factors are labeled assumptions; prefer user-supplied historical velocity/team data over defaults when provided.
- **Re-runs diff.** Summarize what changed vs the previous plan version and why.
- **Self-contained.** All formulas (PERT, communication overhead, AI factors) are defined here so results reproduce across sessions.
- **Evidence-based.** Every WBS item traces to a Phase 1–3 (or `ui_design/`) artifact. No invented pages/services.
- **Non-blocking.** Declining this phase never halts the modernization; the orchestrator proceeds to Phase 4.
- **Commercial-language sanitization.** Any artifact tier a customer will see (e.g. the presentation tier of `director_briefing/`) must not expose internal billing/commitment framing ("recommended commit/billing case", "billing floor", etc.). The internal/presenter tier may keep this language. See [`EXTENDED_ARTIFACTS.md` § 6](./EXTENDED_ARTIFACTS.md#6-commercial-language-sanitization).

---

## Definition of Done

> 📋 **Quality review:** consult [`../quality-playbook/SKILL.md`](../quality-playbook/SKILL.md) when weighing plan trade-offs (monolith vs microservices sequencing, migration strategy) before finalizing the recommendation.

- [ ] All preflight artifacts verified (or missing ones reported and the run halted)
- [ ] Gate 1 completed: constraints & expectations register recorded, every item classified Hard / Preference / Open
- [ ] Conflict check run against `target_architecture.md` and `tech_stack_selections.md`; all conflicts resolved and the resolution recorded
- [ ] Gate 2 completed: candidate plan set presented and explicitly approved (or modified) by the user before estimation
- [ ] ≥2 approved plans fully described and estimated, each citing which constraints it satisfies (by register ID)
- [ ] Frontend page inventory complete — every page traceable to Phase 1/3/4a evidence, each with API consumption listed
- [ ] Service/module inventory complete — every item traceable to target architecture, each with complexity rationale
- [ ] Cross-cutting WBS items included (CI/CD, migration, testing, cutover, etc.)
- [ ] Every WBS item has three-point estimates for both with-AI and without-AI, with the applied productivity factor visible
- [ ] Team-size scenarios computed with critical-path and communication-overhead modeling (not linear division)
- [ ] Scenario matrix present and consistent with per-item estimates (spot-check sums)
- [ ] Dependency DAG and critical path rendered as valid mermaid HTML (passes `node scripts/validate-mermaid.js`)
- [ ] Gantt charts rendered for at least the recommended plan (both AI scenarios)
- [ ] Assumptions register and risk register present
- [ ] Executive summary readable by a non-technical business owner
- [ ] Recommendation with scored decision matrix present (weights derived from Gate 1 A1 ranking)
- [ ] Planning-estimate banner + per-plan confidence level visible in the report
- [ ] On re-run: diff vs previous plan version summarized
- [ ] `CLAUDE.md` roster + orchestrator agent offer Phase 3.5 (framework integration — one-time, verified at install)
- [ ] AI-gain sensitivity band reported (base + stretch, cross-checked against cited external evidence) with a named re-measure gate (`EXTENDED_ARTIFACTS.md` § 1)
- [ ] Step 9.1 optional-artifact triggers evaluated; each triggered artifact produced, each non-triggered one explicitly skipped (not silently omitted)
- [ ] If `block_defensibility_register.md` was triggered: every ≥5-wk block has a DEFEND/TRIM/DECOMPOSE verdict with cited evidence
- [ ] If `open_items_crosswalk.md` was triggered: every carried-forward open item has plan linkage, impact, and owner action
- [ ] If `director_briefing/` was triggered: presenter and customer tiers both produced, customer tier sanitized per `EXTENDED_ARTIFACTS.md` § 6 (verified, not assumed)
