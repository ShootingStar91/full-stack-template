# AI Development Setup — Overview

This document describes the AI-assisted development setup for human readers: how it is structured,
how it is configured for a project, and how the selectable development flows work. The deep-dive
into the specs-driven flow lives in [`full-flow.md`](./full-flow.md).

The setup works with both **Cursor and Claude Code**. Everything of substance lives in the shared
`ai/` directory; the tool-specific files (`.cursorrules`, `CLAUDE.md`, `.cursor/`, `.claude/`) are
thin adapters that point into it. That, plus keeping all project-specific commands in one config
file, makes the setup copyable into any project — including projects with a different stack or
language.

---

## 1. Structure

```mermaid
flowchart TB
    subgraph TOOLS["🔌 Tool adapters (thin pointers)"]
        direction LR
        CUR[".cursorrules · .cursor/skills · .cursor/agents"]
        CLA["CLAUDE.md · .claude/skills · .claude/agents"]
    end

    subgraph AI["📦 ai/ — shared setup (the real content)"]
        RULES["rules.md<br/>session entry rules"]
        PROJ["project.md<br/>active flow · commands ·<br/>migrations policy · doc index"]
        subgraph FLOWS["flows/"]
            SMALL["small-flow.md"]
            FULL["full-flow.md"]
        end
        SKILLS["skills/<br/>9 invocable skills"]
        AGENTS["agents/<br/>3 validator subagents"]
    end

    KNOW["📚 Knowledge docs<br/>server/*.md · client/*.md · git-workflow.md<br/>(indexed in project.md)"]

    CUR --> RULES
    CLA --> RULES
    RULES --> PROJ
    PROJ --> FLOWS
    FLOWS --> SKILLS
    FLOWS --> AGENTS
    FULL --> KNOW
```

| Piece | Role |
|---|---|
| `ai/rules.md` | Read at the start of every session: read `project.md`, follow the active flow, app auto-start rules, git rules. |
| `ai/project.md` | **Single source of truth for project facts**: the active flow, commands for app/tests/lint, the database-migrations policy, and where the knowledge docs live. Every flow and skill defers to it — if a command here conflicts with one written elsewhere, this file wins. |
| `ai/flows/` | The selectable development flows (see below). |
| `ai/skills/` | Instructions for invocable skills (`/run-tests`, `/git-workflow`, `/setup-ai`, spec validation, layout iteration, ...). |
| `ai/agents/` | Instructions for the independent verifier subagents used by the full flow. |
| Tool adapters | Register each skill/agent with Cursor and Claude Code and point at the `ai/` instructions. |

---

## 2. Flows

A **flow** defines how the AI carries out development tasks. The active flow is named in
`ai/project.md`; the user can override it for a single task by asking explicitly, and requests
that clearly use another flow's triggers (e.g. *"let's generate a feature spec"*) follow that
flow.

```mermaid
flowchart LR
    REQ([User request]) --> ENTRY["ai/rules.md<br/>read ai/project.md"]
    ENTRY --> WHICH{Active flow /<br/>explicit trigger?}
    WHICH -->|small-flow| S["ai/flows/small-flow.md"]
    WHICH -->|full-flow| F["ai/flows/full-flow.md"]
```

### small-flow — implement · test · lint · commit

Lightweight flow for small edits, bug fixes, and small features. No specs, no session boundaries,
no validation gates.

```mermaid
flowchart TD
    A([User requests a small change]) --> B{On dev/master?}
    B -->|yes| BR[Branch: feature/*] --> C
    B -->|no| C[Implement, following<br/>existing code conventions]
    C --> D["Run relevant tests<br/>(commands from ai/project.md)"]
    D --> E{Failures?}
    E -->|yes| FIX[Fix them] --> D
    E -->|no| F[Add/update tests if<br/>behavior changed]
    F --> G[Lint + typecheck, fix errors]
    G --> H["Commit (conventions from git-workflow.md)<br/>push only when asked"]
    H --> I([Brief report, wait for next task])
```

If a "small" request turns out to be large, the flow says so and suggests switching to the full
flow instead of silently doing a big change.

### full-flow — specs-driven development

The heavyweight flow: a feature spec is the single source of truth, one task per session, TDD on
the backend, layout-first on the frontend, and mandatory validation gates checked by skills and
independent subagents. Documented in detail in [`full-flow.md`](./full-flow.md).

**Choosing between them**: small-flow suits day-to-day maintenance and teams that find the full
flow too heavy; full-flow suits larger feature work where specs and independent verification pay
off. Teams can also keep small-flow active and invoke the full flow only for selected features.

Adding a new flow = adding a file to `ai/flows/` and mentioning it in `ai/project.md`.

---

## 3. Configuration: the `/setup-ai` Skill

`/setup-ai` (defined in `ai/skills/setup-ai.md`) adapts the setup to a project. It is
**re-runnable**: every step first reports the current state and asks whether to keep or change it.

```mermaid
flowchart TD
    S([User runs /setup-ai]) --> P0["Step 0 · Permissions<br/>detect tool(s) · read existing config<br/>propose safe permissions (git, file edits,<br/>project commands) · user approves · write"]
    P0 --> P1["Step 1 · Commands & migrations<br/>interview: start app? tests? lint?<br/>migrations — how, and may the AI run them?<br/>verify cheap commands · write ai/project.md<br/>propagate to other AI files"]
    P1 --> P2["Step 2 · Flow selection<br/>describe available flows · pick default<br/>apply customizations to ai/flows/*"]
    P2 --> FIN["Final summary<br/>where flows are defined ·<br/>edit directly or ask the AI"]
```

- **Permissions** (step 0): safe git commands (`status`, `diff`, `log`, `branch`, `checkout`,
  `add`, `commit`), reading/editing files in the repo, and the project's test/lint/app commands.
  `git push` and anything touching remote environments always keep prompting. Written to
  `.claude/settings.json` (Claude Code) or `.cursor/cli.json` (Cursor CLI); the user sees and
  approves the exact entries first.
- **Commands & migrations** (step 1): the answers land in `ai/project.md`, and any AI files with
  now-contradicting hardcoded commands are updated. This is the step that makes the setup work in
  a non-template project (different test runner, no Taito CLI, special services).
- **Flow selection** (step 2): sets **Active Flow** in `ai/project.md` and applies any requested
  tweaks (add/remove/change steps) directly to the flow files.

---

## 4. Flow-Independent Skills

### `run-tests`

Saying *"run tests"* runs all suites in order, using the test commands from `ai/project.md`
(template defaults shown), and **auto-starts the app stack when needed** — it never asks the user
to start it.

```mermaid
flowchart TD
    T0([User: run tests]) --> T1["1· Unit tests<br/>(default: taito test-unit)"]
    T1 --> F1{Pass?}
    F1 -->|fail| STOP1[Report & stop]
    F1 -->|pass| T2{App stack running?}
    T2 -->|no| APPSTART["Auto-start<br/>(default: taito start)"]
    T2 -->|yes| T3
    APPSTART --> T3["2· Server integration/e2e tests<br/>(default: taito test:server)"]
    T3 --> F3{Pass?}
    F3 -->|fail| STOP2[Report & stop]
    F3 -->|pass| T5["3· Playwright E2E tests<br/>(default: taito test:playwright)"]
    T5 --> SUM[Summary: pass/fail per suite]
```

### `git-workflow`

The `git-workflow` skill and `git-workflow.md` enforce Angular/commitlint conventions so that
`semantic-release` can auto-generate versions and release notes.

```mermaid
flowchart TD
    G0([User: push to git]) --> G1[git status — check branch]
    G1 --> G2{On dev or master?}
    G2 -->|yes| G3[git checkout -b feature/name dev]
    G2 -->|no| G4
    G3 --> G4["Pre-commit: lint + typecheck · unit tests<br/>(commands from ai/project.md)"]
    G4 --> G5{Errors?}
    G5 -->|yes| G6[Fix before committing] --> G4
    G5 -->|no| G7["git add . → commit<br/>type(scope): subject"]
    G7 --> G8[git push origin current-branch]
```

Commit format: `<type>(<scope>): <subject>` where type ∈ `wip, feat, fix, docs, style, refactor,
perf, test, revert, build, ci, chore`; subject is lowercase, imperative, no trailing period.
`dev` and `master` are **not protected** — work branches off `dev` and merges back via PR.

---

## 5. Using the Setup in Another Project

1. Copy `ai/`, the tool adapters you need (`.cursorrules` + `.cursor/`, and/or `CLAUDE.md` +
   `.claude/`), and `git-workflow.md` into the target repo.
2. Optionally copy the knowledge docs (`server/*.md`, `client/*.md`) if the target follows similar
   patterns — the full flow depends on them, the small flow doesn't. Update the knowledge-doc
   index in `ai/project.md` either way.
3. Run `/setup-ai` there. It replaces the template's commands with the project's own, records the
   migrations policy, sets up permissions, and lets the team pick and customize a flow.

## 6. Customizing

Everything is plain markdown. To change, add, or remove a step in any flow, either edit the file
in `ai/flows/` directly or ask the AI to make the edit. Re-running `/setup-ai` reviews the whole
configuration step by step.
