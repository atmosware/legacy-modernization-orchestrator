# Nexia-Orchestrator

A structured multi-agent framework for end-to-end legacy system modernization.
Works with **Claude Code**, **Cursor**, **OpenAI Codex CLI**, and **GitHub Copilot**.

---

## Agents

Agents are organized by the phase in which they are invoked. The master orchestrator chains them automatically; each agent can also be invoked standalone.

| Phase | Agent / Skill | Role |
|-------|--------------|------|
| Master | `nexia-orchestrator` | Master orchestrator — chains all phases end-to-end |
| 1 | `nexia-legacy-analysis` | Legacy codebase analysis, technical debt, DB schema, integration map, security posture |
| 2 | `nexia-legacy-architecture` | Legacy architecture diagrams — Mermaid HTML, component map, data flow, deployment topology |
| 2.5 | `nexia-tech-stack-selection` | Tech stack selection gate — confirms backend, frontend, DB, mobile, cloud, and infra choices |
| 3 | `nexia-target-architecture` | Target architecture — Clean/Hexagonal/DDD, multi-stack (Java / .NET / Python / Go), API-first design |
| 4a | `nexia-ui-ux-design` | Wireframes, design system, user journeys, WCAG accessibility, HTML previews, design tokens |
| 4b | `nexia-backend-development` | Backend service — REST APIs, JWT/OAuth2, clean architecture, ORM, unit + integration tests, observability |
| 4b+ | `nexia-java-springboot` | Tier-2 — Java 21 + Spring Boot 3, Spring Data JPA, Micrometer, JUnit 5, Testcontainers |
| 4b+ | `nexia-dotnet-aspnetcore` | Tier-2 — .NET 9 + ASP.NET Core, EF Core, Serilog, xUnit, Testcontainers.NET |
| 4b+ | `nexia-python-fastapi` | Tier-2 — Python 3.12 + FastAPI, SQLAlchemy 2, Alembic, Pydantic v2, pytest-asyncio |
| 4b+ | `nexia-go-gin-fiber` | Tier-2 — Go 1.23 + Gin or Fiber, GORM/sqlc, zap/zerolog, testcontainers-go |
| 4c | `nexia-frontend-development` | React / Vue / Angular / Svelte TypeScript — Zustand / Pinia / NgRx, TanStack Query, Vitest, Playwright |
| 4d | `nexia-ios-development` | Swift SwiftUI — MVVM, Combine/async-await, Keychain, CoreData, APNs, XCTest, App Store |
| 4e | `nexia-android-development` | Kotlin Jetpack Compose — MVVM, Coroutines/Flow, Room, Retrofit, FCM, JUnit/Mockk, Play Store |
| 4f | `nexia-data-migration` | Zero-data-loss schema migration — Flyway/Liquibase/Alembic/Goose, dual-write, chunking, rollback |
| 4g | `nexia-security-review` | OWASP Top 10, CVE scanning, JWT/CORS/CSP audit, Docker image security, secrets detection |
| 4h | `nexia-devops-infra` | Kubernetes, Helm, Terraform/Pulumi, GitHub Actions/GitLab CI, Prometheus, Grafana, Vault |
| 4i | `nexia-cross-platform-mobile` | Flutter (Dart) or React Native (TypeScript) — single codebase for iOS + Android, Riverpod/BLoC or Zustand, Detox/integration_test, App Store + Play Store _(optional, non-default)_ |
| 5 | `nexia-compare-legacy-to-new` | Gap analysis, feature-parity check, migration strategy, before/after Mermaid diagrams |
| 6 | `nexia-final-validation` | Release readiness gate — functional completeness, performance baseline, smoke tests, go/no-go |
| Advisory | `nexia-quality-playbook` | Cross-cutting quality reference — architecture patterns, testing strategy, API design, code standards |
| Advisory | `nexia-agent-governance` | Framework governance — agent selection, phase chaining, DoD gates, project resume |

---

## Installation

### Option 1 — npx (recommended)

```bash
npx nexia-orchestrator@latest
```

The interactive prompt asks for scope (global/local) and runtime (Claude Code, Codex, or both).

#### Non-interactive flags

```bash
# Global — available in every project on your machine (recommended)
npx nexia-orchestrator --global

# Local — current project only
npx nexia-orchestrator --local

# Specific runtimes
npx nexia-orchestrator --global --claude   # Claude Code only
npx nexia-orchestrator --global --codex    # Codex CLI only
npx nexia-orchestrator --local  --cursor   # Cursor only (always project-scoped)
npx nexia-orchestrator --global --all      # All runtimes (default)

# Uninstall
npx nexia-orchestrator --uninstall --global --all
```

### Option 2 — Clone and run the shell script

```bash
git clone https://github.com/your-org/legacy-modernization-orchestrator.git
cd legacy-modernization-orchestrator
bash scripts/install.sh --global --all   # Claude Code + Codex CLI + Cursor
```

---

## What gets installed

| Runtime | Location (global) | What |
|---------|------------------|------|
| Claude Code | `~/.claude/agents/*.md` | Named subagents (`@agent-name`) |
| Claude Code | `~/.claude/skills/<name>/SKILL.md` | Slash commands (`/agent-name`) |
| Codex CLI | `~/.codex/skills/<name>/SKILL.md` | Skill commands (`$agent-name`) |
| Cursor | `.cursor/agents.json` + `.cursor/agents/*.md` in the current project | Agent picker entries (`@agent-name` in Cursor chat) |
| GitHub Copilot source | `~/.copilot/{agents,skills}/` if `~/.copilot` exists, otherwise `~/.github/{agents,skills}/` | Canonical agent and skill files used by Copilot, Claude, and Codex |

> For **local** installs, canonical files go into `./.github/{agents,skills}/`, and runtime wrappers go into `./.claude/`, `./.codex/`, and `./.cursor/` inside your project.
>
> **Cursor agents are always project-scoped** — Cursor has no global agent directory. The `--cursor` flag installs into `.cursor/` in the current working directory regardless of `--global` or `--local`.

---

## Usage

### Claude Code

After installing, use agents in two ways:

**As a slash command (skill):**
```
/nexia-orchestrator path/to/my-legacy-app
/nexia-legacy-analysis path/to/my-legacy-app
/nexia-target-architecture MyProject
/nexia-backend-development MyProject
```

**As a subagent (invoked by the orchestrator or directly):**
```
@nexia-orchestrator path/to/my-legacy-app
@nexia-legacy-analysis path/to/my-legacy-app
```

**Typical full workflow:**
```
@nexia-orchestrator path/to/my-legacy-app
```
The orchestrator runs all phases automatically, asking you for scope (which dev targets) before Phase 4.

### Codex CLI

```bash
$nexia-orchestrator path/to/my-legacy-app
$nexia-legacy-analysis path/to/my-legacy-app
$nexia-target-architecture MyProject
$nexia-backend-development MyProject
```

### Cursor

After installing (run `npx nexia-orchestrator --local --cursor` in your project), agents appear in the Cursor agent picker. Reference them in Cursor chat:

```
@nexia-orchestrator path/to/my-legacy-app
@nexia-legacy-analysis path/to/my-legacy-app
@nexia-target-architecture MyProject
@nexia-backend-development MyProject
```

**Typical full workflow:**
```
@nexia-orchestrator path/to/my-legacy-app
```
The nexia-orchestrator runs all phases automatically, asking for scope confirmation before Phase 4.

### GitHub Copilot (VS Code)

Agents are defined in `.github/agents/` for local installs. For global installs, the canonical source is installed under `~/.copilot/agents/` when that folder already exists, otherwise `~/.github/agents/`. Use the agent picker in VS Code Copilot Chat or reference them by name. The framework is also described in `AGENTS.md` at the repo root.

---

## Phase workflow

```
Phase 1    →  nexia-legacy-analysis          (always required)
Phase 2    →  nexia-legacy-architecture      (always required)
Phase 2.5  →  nexia-tech-stack-selection     (always required — user confirms all tech choices)
Phase 3    →  nexia-target-architecture      (always required)
               ↓ ask user: which targets?
Phase 4a   →  nexia-ui-ux-design             (if any client UI; must complete before 4c/4d/4e)
Phase 4b   →  nexia-backend-development      (optional; can run in parallel with 4a)
Phase 4c   →  nexia-frontend-development     (optional; requires 4a)
Phase 4d   →  nexia-ios-development          (optional; requires 4a)
Phase 4e   →  nexia-android-development      (optional; requires 4a)
Phase 4i   →  nexia-cross-platform-mobile    (optional, non-default; Flutter/RN only; mutually exclusive with 4d/4e for same target)
               ↓ 4b/4c/4d/4e/4i can run in parallel with each other (after 4a)
Phase 4f   →  nexia-data-migration           (optional; zero-data-loss schema + data migration)
Phase 4g   →  nexia-security-review          (optional; OWASP Top 10, CVE scan, secrets audit)
Phase 4h   →  nexia-devops-infra             (optional; K8s, Helm, Terraform, CI/CD, observability)
               ↓ 4f/4g/4h can run in parallel with each other (after Phase 3)
Phase 5    →  nexia-compare-legacy-to-new    (after any dev phase)
Phase 6    →  nexia-final-validation         (release readiness gate — go/no-go decision)
```

All outputs are written to `ai-driven-development/` inside your project.

---

## How agents work

Each agent reads its full `SKILL.md` from the canonical source tree before acting. For local installs this is `.github/skills/<name>/SKILL.md`; for global installs this is `~/.copilot/skills/<name>/SKILL.md` when `~/.copilot` exists, otherwise `~/.github/skills/<name>/SKILL.md`. The skill files contain the authoritative step-by-step procedures, output formats, and Definition of Done checklists. Agents never skip, reorder, or summarize steps.

The `.claude/agents/`, `.claude/skills/`, and `.codex/skills/` files installed on your machine are thin wrappers that point back to these skill files.

---

## Repository layout

```
.github/
  agents/          ← GitHub Copilot agent definitions (*.agent.md)
  skills/          ← Authoritative skill SKILL.md files (step-by-step procedures)
  roster.json      ← Single source of truth for the skill/agent roster
  standards/       ← Cross-cutting standards (core.md)
.claude/
  agents/          ← Claude Code subagent definitions (@agent-name)
  skills/          ← Claude Code slash command skills (/agent-name)
.codex/
  skills/          ← Codex CLI skill definitions ($agent-name)
.cursor/
  agents.json      ← Cursor agent registry (id, name, description, path)
  agents/          ← Cursor agent stubs — delegate to .github/agents/*.agent.md
bin/
  install.js       ← npx installer
scripts/
  install.sh       ← Shell installer (alternative)
  uninstall.sh     ← Shell uninstaller
  ci-validate.js   ← Framework self-tests (frontmatter, links, headings, output coverage)
  validate-roster.js ← Roster consistency checks across all source files
  generate-dod.js  ← Regenerate dod.json from SKILL.md DoD sections
  check-adr-prompts.js ← Advisory ADR capture check
  sync-wrappers.js ← Sync .claude/.codex/.cursor wrappers from .github/skills
  new-skill.sh     ← Scaffold a new skill from template
AGENTS.md          ← Loaded by Codex CLI automatically
CLAUDE.md          ← Loaded by Claude Code CLI automatically
CONTRIBUTING.md    ← How to add, edit, or deprecate skills
```

---

## Validation & CI

The framework ships with self-tests that verify structural integrity across all skills, agents, and documentation. Run them before every PR.

```bash
npm run ci:validate       # frontmatter YAML lint, relative link resolution,
                          # required headings (Role / Preflight / DoD), STANDARDS_OUTPUTS coverage

npm run validate:roster   # roster consistency — checks bin/install.js, AGENTS.md, CLAUDE.md,
                          # orchestrator, README.md, STANDARDS_OUTPUTS.md, disk paths,
                          # frontmatter schema, version metadata, deprecation status, dod.json

npm run check:wrappers    # .claude/ and .codex/ wrappers match .github/skills/ sources
npm run check:dod         # dod.json files are in sync with SKILL.md DoD sections
npm run check:adr         # advisory — scans for decision-language that may need an ADR
```

| Script | Enforces | Fails CI? |
|--------|----------|-----------|
| `ci:validate` | (A) Valid YAML frontmatter in every SKILL.md and .agent.md, (B) all relative markdown links resolve, (C) every non-advisory SKILL.md has `## Role` + `## Prerequisites (Preflight)` + `## Definition of Done`, (D) every roster output appears in STANDARDS_OUTPUTS.md | Yes |
| `validate:roster` | Roster.json is the single source of truth — installer, AGENTS.md, CLAUDE.md, orchestrator, README, STANDARDS_OUTPUTS, disk paths, frontmatter schema, version/review dates, deprecation status, dod.json all agree | Yes |
| `check:wrappers` | Runtime wrappers (.claude/.codex) are in sync with authoritative .github/skills/ | Yes |
| `check:dod` | dod.json machine-readable DoD matches SKILL.md markdown DoD | Yes |
| `check:adr` | Flags files with decision-language that may need an ADR (advisory only) | No |

---

## Uninstall

```bash
# npx
npx nexia-orchestrator --uninstall --global --all

# shell
bash scripts/uninstall.sh --global --all
```

---

## License

Apache 2.0
