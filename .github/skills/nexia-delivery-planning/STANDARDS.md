# Delivery Planning Standards

Output formatting, HTML+Mermaid templates, and diagram rules for the `delivery-planning` skill. All HTML files under `ai-driven-development/docs/implementation_planning/` follow the base template below. Every diagram is valid Mermaid embedded in a self-contained HTML file and must pass `node scripts/validate-mermaid.js`.

Use the **warm light design system** — warm off-white/terracotta palette: `#1e3a5f` primary, `#2d6da3` secondary, `#e07840` accent, `#f8f7f5` background, no dark backgrounds anywhere. Mermaid uses `theme: 'default'` with `fontFamily: 'system-ui, Segoe UI, sans-serif'`.

---

## HTML + Mermaid.js Page Template

Base structure for every `*.html` output (`dependency_graph.html`, `gantt_<plan>_<scenario>.html`, `scenario_matrix.html`). Populate the `<pre class="mermaid">` blocks; add sections per file as needed.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Delivery Planning — {DIAGRAM TITLE}</title>
  <script src="https://cdn.jsdelivr.net/npm/mermaid@10/dist/mermaid.min.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/panzoom@9/dist/panzoom.min.js"></script>
  <script>
    mermaid.initialize({
      startOnLoad: false,
      theme: 'default',
      securityLevel: 'loose',
      fontFamily: 'system-ui, Segoe UI, sans-serif',
      fontSize: 15,
      flowchart: { curve: 'basis', htmlLabels: true },
      gantt: { barHeight: 22, barGap: 6, topPadding: 48, leftPadding: 140 }
    });
  </script>
  <style>
    :root {
      --primary: #1e3a5f; --secondary: #2d6da3; --accent: #e07840;
      --green: #3a8a5c; --red: #c0392b; --bg: #f8f7f5;
      --card-bg: #ffffff; --text: #1a1a1a; --border: #e5e0d9; --code-bg: #f3f0eb;
      --crit: #c0392b;
    }
    * { box-sizing: border-box; margin: 0; padding: 0; }
    body { font-family: system-ui, 'Segoe UI', sans-serif; background: var(--bg); color: var(--text); line-height: 1.65; }
    header { background: #f2f5f9; color: #0d2440; padding: 36px 48px; border-bottom: 3px solid var(--accent); }
    header h1 { font-size: 2.2rem; font-weight: 800; letter-spacing: -0.5px; }
    header .subtitle { margin-top: 10px; color: #3a5070; font-size: 1.05rem; }
    .banner { background: #fef6f0; border-left: 4px solid var(--accent); padding: 12px 18px; margin: 20px 48px 0; border-radius: 0 8px 8px 0; font-weight: 600; color: #7a3a12; }
    main { max-width: 1440px; margin: 0 auto; padding: 36px 48px; }
    .section { margin-bottom: 48px; }
    .section h2 { font-size: 1.5rem; color: var(--primary); border-left: 5px solid var(--accent); padding-left: 16px; margin-bottom: 20px; }
    .diagram-wrap { background: var(--card-bg); border: 1px solid var(--border); border-radius: 12px; padding: 24px; margin-bottom: 24px; overflow: hidden; position: relative; min-height: 140px; }
    .diagram-title { font-size: 1.0rem; font-weight: 700; color: var(--primary); margin-bottom: 20px; display: flex; align-items: center; gap: 10px; }
    .diagram-title::before { content: "⬡"; color: var(--accent); font-size: 1.2rem; }
    .mermaid { min-width: 400px; }
    .legend { font-size: 0.85rem; color: #55606c; margin-top: 8px; }
    .legend .crit { color: var(--crit); font-weight: 700; }
    table { width: 100%; border-collapse: collapse; font-size: 0.92rem; background: var(--card-bg); }
    th { background: #eaf0f6; color: #0d2440; padding: 10px 14px; text-align: left; font-weight: 600; }
    td { padding: 8px 14px; border-bottom: 1px solid var(--border); }
    tr:nth-child(even) td { background: #faf8f5; }
  </style>
</head>
<body>
  <header>
    <h1>{DIAGRAM TITLE}</h1>
    <div class="subtitle">{PROJECT} — Implementation Planning (Phase 3.5)</div>
  </header>
  <div class="banner">⚠ Planning estimate — not a commitment. Confidence: ±{X}%.</div>
  <main>
    <div class="section">
      <div class="diagram-wrap">
        <div class="diagram-title">{DIAGRAM LABEL}</div>
        <pre class="mermaid">
{MERMAID DIAGRAM HERE}
        </pre>
        <div class="legend">Legend: <span class="crit">red = critical path</span>; dashed edge = soft/ordering dependency.</div>
      </div>
    </div>
  </main>
  <script>
    (async function() {
      await mermaid.run({ querySelector: '.mermaid' });
      document.querySelectorAll('.diagram-wrap').forEach(function(wrap) {
        var svg = wrap.querySelector('svg');
        if (!svg) return;
        var tb = document.createElement('div');
        tb.style.cssText = 'display:flex;gap:6px;margin-bottom:10px;';
        [['＋','zoomIn'],['－','zoomOut'],['⊙ Reset','reset']].forEach(function(pair) {
          var btn = document.createElement('button');
          btn.textContent = pair[0];
          btn.style.cssText = 'background:#e8eef5;color:#1e3a5f;border:1px solid #2d6da3;border-radius:4px;padding:3px 10px;cursor:pointer;font-size:0.82rem;font-family:inherit;';
          btn.onmouseover = function(){ this.style.background='#e07840'; this.style.color='#fff'; this.style.borderColor='#e07840'; };
          btn.onmouseout  = function(){ this.style.background='#e8eef5'; this.style.color='#1e3a5f'; this.style.borderColor='#2d6da3'; };
          btn._act = pair[1];
          tb.appendChild(btn);
        });
        wrap.insertBefore(tb, wrap.firstChild);
        svg.style.cursor = 'grab';
        var pz = panzoom(svg, { maxZoom: 5, minZoom: 0.15, zoomDoubleClickSpeed: 1 });
        tb.querySelectorAll('button').forEach(function(btn) {
          btn.addEventListener('click', function() { pz[btn._act](); });
        });
      });
    })();
  </script>
</body>
</html>
```

---

## Diagram Type Rules

### Dependency graph — `dependency_graph.html`
- One `<pre class="mermaid">` block, `flowchart LR` (or `TD`).
- **Hard dependency:** solid arrow `A --> B`.
- **Soft/ordering dependency:** dashed arrow `A -.-> B` with an edge label, e.g. `A -.->|prefer| B`.
- **Critical path:** highlight the critical items with a class, e.g.

```
flowchart LR
    S["Scaffold"] --> BE["Auth Service"]
    BE --> API["OpenAPI Stub"]
    API -.->|prefer| FE["Login Page"]
    BE --> FE
    classDef crit fill:#fdecea,stroke:#c0392b,stroke-width:3px,color:#7a1a10;
    class S,BE,FE crit;
```

- State the critical-path length (person-days) in the HTML `.legend` and in `implementation_plan.md`.

### Gantt — `gantt_<plan>_<scenario>.html`
- One `gantt` block per file (Rule 6 — one diagram type per block).
- Header: `dateFormat X` + relative week axis, OR `dateFormat YYYY-MM-DD` only when the user gave a start date.
- With relative weeks, model durations in days and label sections by swimlane. Example:

```
gantt
    title Recommended Plan — with AI (relative weeks)
    dateFormat  X
    axisFormat  W%W
    section Backend Core
    Scaffold + CI/CD        :done, s1, 0, 5d
    Auth Service            :active, a1, after s1, 8d
    section Frontend
    OpenAPI Stub            :st1, after s1, 2d
    Login Page              :l1, after st1, 4d
```

- `{scenario}` in the filename is `withAI` or `noAI`. `{plan}` is the plan slug (e.g. `strangler`, `bigbang`).
- Mark critical-path tasks with the `crit` tag; mark completed baseline setup with `done`.

### Scenario matrix — `scenario_matrix.html`
- Render the plan × team-size × AI comparison as an HTML `<table>` **plus** a mermaid visual (an `xychart-beta` bar comparison of calendar weeks, or a `flowchart` grid if xychart is unavailable in the pinned mermaid 10 build — prefer the HTML table as the source of truth and the mermaid as the visual aid).
- Cells show **calendar weeks + total person-days**. Bold the recommended cell.

---

## Mermaid Syntax Rules & Common Errors

Follow strictly to prevent render failures.

### Rule 1 — Use `<pre class="mermaid">` (not `<div>`)
✅ `<pre class="mermaid">` … `</pre>`  ❌ `<div class="mermaid">` (browser may HTML-parse before Mermaid runs).

### Rule 2 — Line breaks in node labels
Use `<br/>` inside quoted labels. Never `\n`.

### Rule 3 — Node IDs and edge labels
Node IDs contain no spaces or reserved keywords (`end`, `graph`, `class`). Quote any label containing `()`, `/`, `?`, `,`, `:` or multiple words: `A["Auth Service (JWT)"]`.

### Rule 4 — Gantt task text
Avoid `:` inside a task name (it is the field separator). Quote or rephrase. Durations use `Nd` (days) or `Nw` (weeks); with `dateFormat X`, use integer offsets + `after <id>`.

### Rule 5 — Always close blocks
Every `subgraph` closes with `end`; every `section` in gantt is followed by tasks.

### Rule 6 — One diagram type per block
Each `<pre class="mermaid">` contains exactly one diagram. Valid openers here: `flowchart LR`, `flowchart TD`, `graph TB`, `gantt`, `xychart-beta`.

---

## File Creation Validation Checklist

After writing each HTML file with the runtime's file-writing mechanism, verify before marking the step complete:

1. **File exists** at the exact path under `ai-driven-development/docs/implementation_planning/`.
2. **Single HTML document** — `<!DOCTYPE html>` appears exactly **once**.
3. **Use `<pre class="mermaid">`** — no `<div class="mermaid">` anywhere.
4. **Tag balance** — every `<pre class="mermaid">` has a matching `</pre>` on its own line.
5. **Diagram count** matches the planned number of diagrams.
6. **No `\n` in labels** — use `<br/>`.
7. **Quoted special chars** — labels/task names with `()`, `/`, `?`, `,`, `:`, or multiple words are double-quoted.
8. **All blocks closed** — every `subgraph`/`section` is well-formed; gantt tasks reference valid ids.
9. **No empty diagram blocks** — each block has real content.
10. **`node scripts/validate-mermaid.js` exits 0** (run `--fix` first if needed).

If any check fails, **overwrite the entire file** with corrected content — never patch individual lines.

---

## Report Table Templates

### Constraints & Expectations Register
```
| ID | Question | Answer | Class (Hard/Pref/Open) | Source of default | Conflict / Resolution | Notes |
```

### Frontend page inventory
```
| Page | User journey | Complexity | APIs consumed (service · endpoint · method) | Reused DS components | Effort no-AI (E, days) | Effort with-AI (E, days) | AI factor | Depends on | Source |
```

### Service/module inventory
```
| Service/Module | Responsibilities | Endpoints | Integration points | Migration involvement | Complexity + rationale | Effort no-AI (E) | Effort with-AI (E) | AI factor | Depends on | Source |
```

### Scenario matrix (markdown mirror of `scenario_matrix.html`)
```
| Plan | 2 devs no-AI | 2 devs +AI | 4 devs no-AI | 4 devs +AI | 6 devs no-AI | 6 devs +AI |
|------|--------------|------------|--------------|------------|--------------|------------|
|      | Xw / Yd      | Xw / Yd    | …            | …          | …            | …          |
```
(`Xw` = calendar weeks, `Yd` = total person-days.)

### Decision matrix (recommendation)
```
| Criterion (weight from Gate 1 A1) | Weight | Plan A | Plan B | Plan C |
| Time-to-first-value | w1 | … | … | … |
| Total duration | w2 | … | … | … |
| Risk | w3 | … | … | … |
| Cost proxy | w4 | … | … | … |
| Business disruption | w5 | … | … | … |
| **Weighted total** | | | | |
```

### Assumptions register
```
| # | Assumption | Value | Source (default / user) | Impact if wrong |
```

### Risk register
```
| # | Risk | Likelihood | Impact | Affected plan(s) | Mitigation |
```
