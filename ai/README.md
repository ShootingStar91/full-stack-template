# AI Development Setup

This directory holds the AI-assisted development setup, shared across AI tools (Cursor, Claude
Code, ...). The tool-specific files (`.cursorrules`, `CLAUDE.md`, `.cursor/`, `.claude/`) are thin
adapters that point here — the actual content lives in this directory so it is written once and
works everywhere.

## Structure

| Path | What it is |
|---|---|
| `rules.md` | Session entry point — rules that apply to every AI session. |
| `project.md` | **Project configuration**: active flow, commands (app, tests, lint), migrations policy, knowledge-doc locations. The single source of truth for project-specific facts. |
| `flows/` | Selectable development flows. `small-flow.md` (implement → test → lint → commit) and `full-flow.md` (specs-driven with validation gates). The active one is named in `project.md`. |
| `skills/` | Instructions for invocable skills (`/run-tests`, `/git-workflow`, `/setup-ai`, ...). |
| `agents/` | Instructions for verifier subagents used by the full flow. |
| `docs/` | Human-facing documentation with diagrams: [`docs/overview.md`](docs/overview.md) for the whole setup, [`docs/full-flow.md`](docs/full-flow.md) for the specs-driven flow in depth. |

Knowledge docs (architecture, testing patterns, implementation flows) live next to the code they
describe (`server/*.md`, `client/*.md`) and are indexed in `project.md`.

## Getting started

Run the **`/setup-ai`** skill. It walks through tool permissions, project commands (including how
database migrations are handled), and flow selection. It is safe to re-run anytime — it reports
the current configuration and asks what to change.

To adjust a flow later, edit the file in `flows/` directly or ask the AI to do it.

## Copying this setup to another project

The setup is designed to be portable, including to projects with different stacks and languages:

1. Copy the `ai/` directory, the tool adapters you need (`.cursorrules` + `.cursor/`, and/or
   `CLAUDE.md` + `.claude/`), and `git-workflow.md` into the target repo.
2. Optionally copy the knowledge docs (`server/*.md`, `client/*.md`) if the target project follows
   similar patterns — otherwise skip them and update the "Knowledge docs" table in `project.md`
   (the full flow depends on them; the small flow doesn't).
3. Run `/setup-ai` in the target project. It replaces the template's Taito CLI commands with the
   project's own, records the migrations policy, sets up permissions, and lets the team pick and
   customize a flow.
