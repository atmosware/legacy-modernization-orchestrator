# Delivery Planning — Extended Artifacts & Techniques

Optional/conditional techniques and artifacts for the `delivery-planning` skill. `SKILL.md` links here for the full method; this file is only loaded when one of the trigger conditions below is met. Nothing here changes the two-gate flow, the preflight gate, or the core PERT/team-scaling formulas in `SKILL.md` — these are additive.

**Contents**
1. [AI-gain sensitivity band](#1-ai-gain-sensitivity-band)
2. [Portion-split estimation](#2-portion-split-estimation)
3. [Block Defensibility Register](#3-block-defensibility-register)
4. [Open-Item Crosswalk](#4-open-item-crosswalk)
5. [Customer-facing presentation deck](#5-customer-facing-presentation-deck)
6. [Commercial-language sanitization](#6-commercial-language-sanitization)

---

## 1. AI-gain sensitivity band

**Trigger:** always, once the per-category AI factor rollup (§ Estimation Model, item 2 in `SKILL.md`) produces a program-wide blended reduction. Report a band, not a single point.

**Why:** a bottom-up per-category rollup can land well above what published field evidence supports. Presenting only the bottom-up number invites a customer or reviewer to discover the gap themselves and lose trust in the whole estimate.

**Method:**
1. Compute the bottom-up blended reduction (PERT-weighted average of the per-category factors actually applied across the WBS) — this is the **stretch** point.
2. Cross-check it against externally published productivity evidence for AI-assisted coding (task-level studies, field trials, RCTs on mature codebases). Cite the source class and figures (do not fabricate citations; if none are available, say so explicitly).
3. If the bottom-up stretch figure exceeds the field-trial range, do not silently keep the higher number. Set a **base** case at or near the field-trial midpoint and present both:
   - **Base** — the planning/commitment figure.
   - **Stretch** — the bottom-up ceiling, labeled "optimistic ceiling, not the expected case."
4. State the band in `implementation_plan.md` § Estimation Model and carry both figures into the Scenario Matrix as separate columns or a clearly labeled note (do not silently pick one for the matrix cells).

**Re-measure gate (binding — record as a numbered assumption and as a risk-register row):**
- Name the exact trigger point: e.g. "after the first N pilot WBS items across the categories with the largest weight" (default: first 2 backend modules + first frontend cluster, or 10% of total WBS person-days, whichever is smaller).
- State what happens at the gate: replace the assumed factor with observed velocity and re-issue the band.
- State the fallback if observed velocity underperforms: which WBS units revert to which factor, and the resulting effort delta (show the arithmetic, e.g. "unit X 0.43→0.54 pd → +N pd").

**Template (add to `implementation_plan.md`):**
```
> ⚠ How to read the with-AI numbers (base vs stretch).
> Bottom-up blended reduction: {STRETCH}% — optimistic ceiling, not the expected case.
> External evidence: {cite 2–3 published figures/ranges with source class}.
> Base (planning/commitment) case: {BASE}%.
> Re-measure gate: after {trigger}, replace the assumed factor with observed velocity; re-issue this band.
> If unrealized: revert {unit} {old}→{new}, {+N pd} to {scenario}.
```

---

## 2. Portion-split estimation

**Trigger:** a single WBS row spans sub-work with materially different business-rule density, novelty, or integration load (e.g. one "module" row is 80% mechanical passthrough and 20% genuinely novel logic). A flat per-category AI factor applied to the whole row either over- or under-states the achievable reduction.

**Why:** the default model (§ Estimation Model, item 2 in `SKILL.md`) applies one category factor per WBS row. That is correct when a row is internally homogeneous. When it is not, decompose it rather than force-fit one factor — but only when the heterogeneity is real and evidenced, not as a way to manufacture a more favorable number.

**Method:**
1. Split the row's O/M/P into named sub-portions that sum back to the original three-point estimate exactly (the split must not change the row's PERT E on its own — if it does, that is a re-estimation and must be labeled and justified separately, not folded silently into the split).
2. Assign each sub-portion its own category (from the § Estimation Model table) and AI factor.
3. Compute each sub-portion's with-AI effort, then sum for the row's with-AI total.
4. Cite the rationale for the split ratio (why 80/20, not 70/30) — a ratio without cited rationale is not acceptable per the Complexity Model's "never record a rating as a bare number" rule.
5. Record the split in the relevant inventory row's Source column and reference the fuller reasoning from `implementation_plan.md`.

**Guardrail:** a portion split that changes the row's total PERT E is a re-estimation, not a split — flag it as such (e.g. "re-baseline: was X, now Y, because Z") rather than presenting it as a neutral decomposition.

---

## 3. Block Defensibility Register

**Trigger:** the business owner is billed/planned by effort and the recommended plan's gantt contains one or more blocks at or above a size threshold (default: **≥ 5 work-weeks / ≈25 wd** on the recommended with-AI gantt). Produce `block_defensibility_register.md` when this condition holds; otherwise omit it (no empty placeholder file).

**Why:** a business owner scanning a gantt does not challenge individual days — they challenge visually large blocks. This register pre-answers, per block, "why does reproducing existing functionality take this long?" before the question is asked.

**Framing:** state explicitly which modernization mode applies (e.g. like-for-like: keep behavior, keep schema) — that framing determines which cost drivers are legitimately defensible and which are legitimately trimmable. Judge every block against that stated framing, not a generic complexity narrative.

**Schema:**
```
| # | Block | Role | With-AI pd (gantt wd) | No-AI pd | Verdict | Why it costs this much |
|---|---|---|---|---|---|---|
```
- **Verdict** — one of:
  - `DEFEND` — size is caused by legacy complexity/behavior that must be reproduced; hold the number.
  - `TRIM` — the stated framing does not justify the full size; a concrete, owner-confirmable reduction lever exists (cite it — e.g. a scope-reduction option already in `implementation_plan.md` § Recommendation).
  - `DECOMPOSE` — size is legitimate but must be shown as sub-parts (see § Portion-split above) to be believed.
- Every row cites concrete evidence (LOC, table/column counts, integration count, god-class names) from Phase 1–3 artifacts — no unsupported size claims.
- List sub-threshold blocks in a single summary line at the end ("reads as normal-length, grouped defense only") rather than individual rows.
- If any gantt bar merges multiple WBS items into one visual row (a layout choice, not a real single task), add a short section explaining the merge so block length is not misread as one task.

**Companion note (required at the top of the file):** state clearly that this document changes no estimate — it is a defense aid over the numbers already in `implementation_plan.md`.

---

## 4. Open-Item Crosswalk

**Trigger:** the plan carries forward Phase 1 open questions and/or Phase 3 open design points (ODPs) whose resolution could change the schedule, **and** the business owner needs each item's schedule impact and owner action made explicit (not just listed) — e.g. after an external/customer review asks "what exactly happens if each of these isn't resolved in time?" Otherwise, the Assumptions Register in `implementation_plan.md` (§ Estimation Model, item 5) is sufficient and this file may be omitted.

**Why:** an assumptions register captures *estimation* assumptions. It does not, by itself, map every carried-forward *open question* to where it lives in the plan, what it blocks, and who must act. When a reviewer asks for that traceability, provide it as its own artifact rather than expanding the assumptions register past its purpose.

**Schema:**
```
| Item | Impact (🔴 High / 🟠 Med / 🟢 Low / ⚪ Ops) | One line |
```
plus, per source category (Phase 1 open questions, Phase 3 ODPs), a detailed table:
```
| # | Item (abridged) | Plan linkage (§ or file) | Impact | Owner action / status |
```
- **Impact key** — 🔴 can move the critical path ~1:1 · 🟠 can add effort/elapsed to a stream · 🟢 little/no effort impact if answered on time · ⚪ operational/cost only, no app-team effort.
- Lead with a short "top schedule-impact items" table (🔴 and the highest 🟠 items) so the reader does not have to scan the full crosswalk to find what matters.
- Every row must resolve to a real Phase 1/3 artifact reference — do not invent open items, and do not resolve any item "by guessing" (this is a hard guardrail — see `SKILL.md` § Guardrails).

---

## 5. Customer-facing presentation deck

**Trigger:** the business owner (or an intermediary presenting on their behalf) needs a director/executive-level visual readout of the plan, distinct from the markdown reports and HTML diagrams that `SKILL.md` already requires. This is genuinely optional — many engagements never need it.

**Structure (adapt section count to the approved plan set; do not invent sections beyond what the underlying reports support):**
1. **Overview / intro** — what the modernization is, in one screen, not a bare title slide.
2. **Current state** — legacy architecture snapshot, complexity scorecard, top risks, sourced from `legacy_analysis.md` / `legacy_architecture.md`.
3. **Target strategy** — the approved target architecture and why (Gate 2 rationale), sourced from `target_architecture.md` and § 2 of `implementation_plan.md`.
4. **One slide (or slide pair) per approved plan** — team shape, effort/calendar with and without AI (base **and** stretch, per § 1 above), role/person breakdown.
5. **Executive recommendation** — the decision matrix outcome from `implementation_plan.md` § Recommendation, plan confidence, the planning-estimate banner.
6. **Appendix — abbreviations/systems glossary** — every acronym and external system named in the deck, defined. Do not leave an abbreviation unexplained on a director-facing slide.

**Two-tier requirement (mandatory whenever this artifact is produced):**
- **Presenter/internal tier** — full content, may include speaker notes, internal planning language (see § 6 below), and reasoning detail.
- **Customer/presentation tier** — the version that will actually be shown to or read by the business owner or their customer. Strip: speaker notes, internal navigation chrome not needed live, reader-oriented metadata (e.g. "N min read" — it will be presented, not read), and all commercially sensitive language (§ 6).
- Never ship only the presenter tier when the deck's stated purpose is external presentation — the sanitization step is not optional once this artifact is triggered.

**Format:** self-contained HTML (matching the visual system used for the plan's other HTML outputs where practical) plus a print-accurate PDF export. If a PDF is produced, verify page boundaries do not split a slide's content across two pages before treating the export as done — check page count and per-page content, not just that a file was written.

**Placement:** `ai-driven-development/docs/implementation_planning/director_briefing/` (or an equivalently named subdirectory) — keep it under the standard output directory, not scattered elsewhere.

---

## 6. Commercial-language sanitization

**Trigger:** whenever the customer/presentation tier of any artifact (§ 5, or any other document a business owner's customer will see) is produced.

**Rule:** the customer-facing tier must not expose internal billing/commitment framing. Concretely, avoid or rewrite:
- "recommended commit/billing case" → "recommended (planning) case"
- "billing floor" → "conservative floor" (or similar neutral framing)
- "commit to X% AI relief" → "plan at X% AI relief"
- any phrase implying the estimate is a contractual commitment, contradicting the mandatory planning-estimate banner (`SKILL.md` § Guardrails: "never present estimates as commitments")

The **presenter/internal tier keeps this language** — the rule applies only to what the customer will see. Before marking a customer-facing artifact done, grep it for "billing", "commit", and any other commercially sensitive terms specific to the engagement, and confirm zero unintended matches.
