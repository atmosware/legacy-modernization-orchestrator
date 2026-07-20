# Legacy Modernization Orchestrator — Claude Instructions

This repository contains a structured multi-agent framework for end-to-end legacy system modernization. You must use the agents and skills defined here whenever a user asks you to analyse, design, or modernize a legacy system.

---

## Repository Layout

```
.github/
  agents/    ← Agent definitions (*.agent.md) — read these for role and DoD rules
  skills/    ← Detailed skill instructions (**/SKILL.md) — read these before doing any work
ai-driven-development/   ← All generated outputs go here (created during runs)
```

---

## Core Principle

When invoked for any legacy modernization task, you MUST:
1. Identify the correct agent from the roster below.
2. **Read the agent's `SKILL.md` file in full** before starting.
3. Follow every step in the skill exactly — no skipping, reordering, or summarizing.
4. Validate the Definition of Done checklist before declaring an agent complete.

---

## Agent Roster

### Master Orchestrator: `nexia-orchestrator`
**Trigger:** Starting or continuing a full legacy modernization project end-to-end.  
**Agent file:** `.github/agents/nexia-orchestrator.agent.md`  

**Mandatory phase sequence:**

| Phase | Agent | Required? |
|-------|-------|-----------|
| 1 | `nexia-legacy-analysis` | Always |
| 2 | `nexia-legacy-architecture` | Always |
| 2.5 | Tech Stack Selection Gate | Always |
| 3 | `nexia-target-architecture` | Always |
| 3.5 | `nexia-delivery-planning` | Optional — offered after Phase 3 DoD passes; never blocking |
| 4a | `nexia-ui-ux-design` | If any client UI needed |
| 4b | `nexia-backend-development` | Optional |
| 4c | `nexia-frontend-development` | Optional |
| 4d | `nexia-ios-development` | Optional |
| 4e | `nexia-android-development` | Optional |
| 4f | `nexia-data-migration` | Optional |
| 4g | `nexia-security-review` | Optional |
| 4h | `nexia-devops-infra` | Optional |
| 4i | `nexia-cross-platform-mobile` | Optional (non-default) |
| 5 | `nexia-compare-legacy-to-new` | After any dev phase |
| 6 | `nexia-final-validation` | After Phase 5 |

> Phase 4a must complete before 4c/4d/4e/4i (they depend on wireframes); 4a can run in parallel with 4b; 4b/4c/4d/4e/4i can run in parallel with each other as scope allows. **4i is mutually exclusive with 4d/4e for the same mobile target** — do not run both native and cross-platform for the same platform. Before entering Phase 4, present the auto-detected development targets from Phase 1 for confirmation; ask directly only if Phase 1 is unavailable or ambiguous.
>
> **Phase 3.5 (`nexia-delivery-planning`) is optional and never blocking.** After Phase 3 DoD passes, the orchestrator **offers** it; if the business owner declines, proceed directly to Phase 4. It may run any time after Phase 3 and before or during Phase 4, and supports **standalone re-run** to refresh the plan when scope or tech stack changes. It reads Phase 1–3 artifacts (and `ui_design/` if Phase 4a already ran) and produces plans/estimates only — it never changes target-architecture decisions or gates Phase 4.

---

### `nexia-legacy-analysis`
Legacy system analysis — reverse engineering, technical debt, business flows, DB schema, integration maps, security posture.  
**Skill:** `.github/skills/nexia-legacy-analysis/SKILL.md`  
**Output dir:** `ai-driven-development/docs/legacy_analysis/`

---

### `nexia-legacy-architecture`
Legacy architecture visualization — component diagrams, data flow maps, mermaid HTML diagrams, architectural constraint documentation.  
**Skill:** `.github/skills/nexia-legacy-architecture/SKILL.md`  
**Output dir:** `ai-driven-development/docs/legacy_architecture/`

---

### `nexia-target-architecture`
Target architecture design — Clean/Hexagonal/DDD patterns, service boundaries, API-first design, user-selected: Java/.NET/Python/Go backend, React/Vue/Angular/Svelte frontend, Kotlin mobile stack.  
**Skill:** `.github/skills/nexia-target-architecture/SKILL.md`  
**Output dir:** `ai-driven-development/docs/target_architecture/`

---

### `nexia-delivery-planning` _(Phase 3.5 — Optional, non-blocking)_
Implementation planning & effort estimation for business owners — alternative delivery strategies, effort and calendar-time estimates under different team sizes (with and without agentic AI tooling), full work breakdown with dependencies, and parallelizable work streams. Runs a **constraints-first two-gate flow**: Gate 1 elicits business constraints/expectations (classified Hard/Preference/Open) and cross-checks them against the target state; Gate 2 proposes 2–4 candidate plans for approval; only then does estimation run. Offered after Phase 3 DoD passes — the business owner may decline. Supports standalone re-run to refresh the plan when scope or tech stack changes.  
**Trigger:** Runnable any time after Phase 3 completes, before or during Phase 4.  
**Skill:** `.github/skills/nexia-delivery-planning/SKILL.md`  
**Output dir:** `ai-driven-development/docs/implementation_planning/`  
**Prerequisites:** `legacy_analysis/legacy_analysis.md`, `legacy_architecture/legacy_architecture.md`, `tech_stack_selections.md`, and `target_architecture/target_architecture.md` must exist. `ui_design/` is optional — when absent, the page inventory is derived from legacy analysis + target architecture and marked "derived, pending UI/UX confirmation".

---

### `nexia-ui-ux-design`
UI/UX design — wireframes, design system, user journeys for web and mobile, WCAG accessibility, HTML design previews, component tokens.  
**Skill:** `.github/skills/nexia-ui-ux-design/SKILL.md`  
**Output dir:** `ai-driven-development/docs/ui_design/`

---

### `nexia-backend-development`
Java Spring Boot / .NET ASP.NET Core / Python FastAPI / Go Gin-Fiber backend — clean/hexagonal architecture, DDD modules, REST APIs, JWT/OAuth2, ORM repositories, unit/integration/Testcontainers testing, observability.  
**Skill:** `.github/skills/nexia-backend-development/SKILL.md`  
**Prerequisites:** `target_architecture/target_architecture.md` must exist.

---

### `nexia-frontend-development`
React / Vue / Angular / Svelte TypeScript frontend — design system components, TanStack Query / Zustand / Pinia / NgRx state management, Axios, Vitest/Playwright testing. Use `nexia-ios-development` or `nexia-android-development` for mobile.  
**Skill:** `.github/skills/nexia-frontend-development/SKILL.md`  
**Prerequisites:** UI/UX design artifacts and system design must exist.

---

### `nexia-ios-development`
Swift SwiftUI iOS app — MVVM, Combine/async-await, Keychain, URLSession, CoreData, FCM, XCTest, App Store deployment.  
**Skill:** `.github/skills/nexia-ios-development/SKILL.md`  
**Prerequisites:** UI/UX design artifacts and system design must exist.

---

### `nexia-android-development`
Kotlin Jetpack Compose Android app — MVVM Clean Architecture, Coroutines Flow, EncryptedSharedPreferences/Keystore, Retrofit/OkHttp, Room, FCM, JUnit/Mockk/Turbine, Play Store deployment.  
**Skill:** `.github/skills/nexia-android-development/SKILL.md`  
**Prerequisites:** UI/UX design artifacts and system design must exist.

---

### `nexia-cross-platform-mobile` _(Phase 4i — Optional, non-default)_
Flutter (Dart) or React Native (TypeScript) cross-platform mobile app targeting iOS and Android from a single codebase — Riverpod/BLoC or Zustand/Redux Toolkit state management, secure storage, Dio/Axios networking, FCM, flutter integration_test or Detox E2E testing, App Store and Play Store deployment.  
**Skill:** `.github/skills/nexia-cross-platform-mobile/SKILL.md`  
**Output dir:** `ai-driven-development/development/mobile_development/cross-platform/`  
**Prerequisites:** UI/UX design artifacts, system design, and `tech_stack_selections.md` confirming Flutter or React Native must exist.

---

### `nexia-compare-legacy-to-new`
Gap analysis — legacy vs new system comparison, migration strategy, before-after mermaid HTML diagrams, regression and improvement identification.  
**Skill:** `.github/skills/nexia-compare-legacy-to-new/SKILL.md`  
**Prerequisites:** Legacy analysis and at least one development phase complete.

---

### `nexia-data-migration`
Zero-data-loss schema migration — Flyway/Liquibase/Alembic/Goose scripts, dual-write reconciliation, large-table chunking, row-count/checksum validation, data cleansing pipelines, cutover freeze SQL and rollback procedures.  
**Skill:** `.github/skills/nexia-data-migration/SKILL.md`  
**Output dir:** `ai-driven-development/development/data_migration/`  
**Prerequisites:** `legacy_analysis/legacy_analysis.md`, `target_architecture/target_architecture.md`, and `tech_stack_selections.md` must exist.

---

### `nexia-security-review`
OWASP Top 10 checks per layer — hardcoded-secret detection, dependency CVE scanning (OWASP Dependency-Check / Trivy), API authorization audit, JWT validation review, CORS/CSP configuration, Docker image security.  
**Skill:** `.github/skills/nexia-security-review/SKILL.md`  
**Output dir:** `ai-driven-development/docs/security_review/`  
**Prerequisites:** `target_architecture/target_architecture.md`, `tech_stack_selections.md`, and any Phase 4 code outputs in scope must exist.

---

### `nexia-devops-infra`
Kubernetes manifests, Helm charts, Terraform/Pulumi modules, GitHub Actions / GitLab CI pipelines, Prometheus alerting rules, Grafana dashboards, secret management (HashiCorp Vault / External Secrets Operator), Docker image hardening.  
**Skill:** `.github/skills/nexia-devops-infra/SKILL.md`  
**Output dir:** `ai-driven-development/development/infra/`  
**Prerequisites:** `target_architecture/target_architecture.md`, `tech_stack_selections.md`, and backend code artifacts must exist.

---

### `nexia-final-validation`
Release readiness gate — functional completeness check, performance baseline, security clearance, operational readiness review, smoke test plan, go/no-go decision.  
**Skill:** `.github/skills/nexia-final-validation/SKILL.md`  
**Output dir:** `ai-driven-development/docs/final_validation/`  
**Prerequisites:** Phase 5 comparison report (`legacy_vs_new_system/compare_legacy_to_new_system.md`), `target_architecture/target_architecture.md`, and all in-scope Phase 4 todo files must exist.

---

### Tier-2 Backend Language Skills

These skills supplement `nexia-backend-development` with language-specific implementation patterns. Apply the matching skill **after** `tech_stack_selections.md` confirms the backend language choice. They share the same output directory as `nexia-backend-development`.

| Skill | When to apply | Skill file |
|---|---|---|
| `nexia-java-springboot` | Java 21 + Spring Boot 3 confirmed | `.github/skills/nexia-java-springboot/SKILL.md` |
| `nexia-dotnet-aspnetcore` | .NET 9 + ASP.NET Core confirmed | `.github/skills/nexia-dotnet-aspnetcore/SKILL.md` |
| `nexia-python-fastapi` | Python 3.12 + FastAPI confirmed | `.github/skills/nexia-python-fastapi/SKILL.md` |
| `nexia-go-gin-fiber` | Go 1.23 + Gin or Fiber confirmed | `.github/skills/nexia-go-gin-fiber/SKILL.md` |

**Output dir (all four):** `ai-driven-development/development/backend_development/`  
**Prerequisites:** `target_architecture/target_architecture.md` and `tech_stack_selections.md` must exist.

---

### `nexia-quality-playbook`
Cross-cutting quality reference — architecture decision trees (monolith vs microservices, DB strategy), design pattern selection, testing strategy, code quality standards, API design evaluation. Consult at any phase; produces no standalone output artifact.  
**Skill:** `.github/skills/nexia-quality-playbook/SKILL.md`  
**Output dir:** N/A — advisory reference only; findings are embedded in the calling phase's output.  
**Prerequisites:** None — consult at any phase.

---

### `nexia-agent-governance`
Framework governance reference — agent selection, phase chaining rules, DoD gate enforcement, standalone vs full-orchestration mode selection, project resume procedure.  
**Skill:** `.github/skills/nexia-agent-governance/SKILL.md`  
**Output dir:** N/A — governance reference only.  
**Prerequisites:** None — consult when unsure which agent to invoke or how to resume a project.

---

### `nexia-tech-stack-selection` _(Phase 2.5 Gate)_
Collects all flexible technology choices (backend language, frontend framework, database, mobile targets, cloud provider, secret manager, deployment platform) after Phase 2 and writes them to `tech_stack_selections.md`. All downstream agents (Phases 3–4) read from this file exclusively — do **not** ask for tech choices again after this gate.  
**Skill:** `.github/skills/nexia-tech-stack-selection/SKILL.md`  
**Template / output schema:** `.github/skills/nexia-tech-stack-selection/tech_stack_selections.template.md`  
**Output:** `ai-driven-development/docs/tech_stack_selections.md`
**Prerequisites:** `legacy_architecture/legacy_architecture.md` must exist (Phase 2 complete).

---

## Execution Rules

1. **Read the full `SKILL.md` first** — every time, for every agent invocation.
2. **Phases 1–3 are mandatory** — never skip analysis and design phases.
3. **No partial DoD** — all checklist items must be ✅ before the phase is considered done.
4. **Output location matters** — write all artifacts to the directories specified in the skill.
5. **Auto-detect scope from Phase 1** — after `nexia-legacy-analysis` completes, read **Section 10 — Technology Profile** in `legacy_analysis.md` to pre-fill scope (Backend / Web Frontend / iOS / Android / Cross-Platform Mobile). If mobile is detected, also use the recorded `Mobile Framework` field to distinguish native vs Flutter vs React Native before Phase 4. Present the detected scope to the user for confirmation. Do NOT ask all 4 questions blindly when the profile is already available.
6. **Phase 2.5 (Tech Stack Selection Gate) is mandatory** — collect ALL flexible technology choices from the user after Phase 2, save to `ai-driven-development/docs/tech_stack_selections.md`. All downstream agents read from this file. Do NOT ask for tech choices again in Phases 3–4.
7. **Only execute phases relevant to the confirmed scope** — skip and mark N/A any Phase 4 sub-phase whose tier is absent from the repository. Do not design, diagram, or produce code for layers that don't exist.
8. **Evidence-based only** — no assumptions; findings must be backed by code, config, or schema evidence.
9. **Choose dependencies from stable, supported release lines** — prefer LTS or current stable releases, avoid prerelease channels (`beta`, `rc`, `canary`, `preview`, `next`) unless explicitly requested, and never invent exact package versions without verifying them from project files or authoritative release sources available in the runtime.
