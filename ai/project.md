# AI Project Configuration

This file is the single source of truth for **project-specific** facts the AI needs: how to run
the app and tests, how database migrations are handled, and which development flow is active.
Flows and skills reference this file instead of hardcoding commands, so the whole `ai/` setup can
be copied into any project and adapted by editing only this file.

> **Status: template defaults.** These values are the defaults for projects created from
> full-stack-template (Taito CLI based). Run the `/setup-ai` skill to review and customize them
> for this project — or edit this file directly.

## Active Flow

**Active flow: `full-flow`**

The active flow defines how the AI works on development tasks. Available flows are defined in
`ai/flows/`:

- `small-flow` — lightweight: implement the requested change, run and fix tests, lint, commit.
- `full-flow` — specs-driven: feature specs as source of truth, TDD, validation gates, subagent
  verification.

The user can always override the active flow for a single task by explicitly asking (e.g. *"use
the small flow for this"*). Change the default by editing the line above or re-running `/setup-ai`.

## Commands

All AI flows and skills must use these commands. If a command listed here conflicts with one
mentioned elsewhere in the AI docs, this file wins.

### Running the app

"Running the app" means the full stack (client, server, database, etc.) in dev mode.

| Purpose | Command |
|---|---|
| Start the full stack | `taito start` (run in background) |
| Clean start with freshly initialized database | `taito start --clean --init` |
| Check whether the server is up (health check) | `taito curl server` |
| Get the client URL | `taito link client` (default `http://localhost:9999/`) |
| Stop the stack | `taito stop` |

The AI starts the app automatically when a task needs it (tests, layout iteration, live checks) —
it never asks the user to start it. Poll the health check with short waits until the stack is up.

### Tests

| Purpose | Command | Requires running stack? |
|---|---|---|
| All unit tests (client + server) | `taito test-unit` | no |
| Client unit tests only | `taito test-unit:client` | no |
| Server unit tests only | `taito test-unit:server` | no |
| Server integration & e2e tests | `taito test:server` | yes |
| Playwright E2E tests | `taito test:playwright` | yes |
| All integration + E2E suites | `taito test` | yes |

### Lint & typecheck

| Purpose | Command |
|---|---|
| Lint + typecheck everything | `taito lint` |

### Database migrations

- **How migrations are made:** migration files live in the server; they are applied with
  `taito db deploy` (locally this runs `npm run db:migrate && npm run db:seed` in the server
  container). A clean start (`taito start --clean --init`) also applies them.
- **May the AI create and apply migrations itself?** **yes** (local environment only). The AI may
  create migration files and apply them locally as part of implementing a feature. It must never
  run migrations against remote environments.

## Git

- Conventions (branching, commit format) are defined in `git-workflow.md` and executed via the
  `git-workflow` skill.
- The AI may branch, stage, and commit as part of a flow. It only pushes when the user asks.

## Knowledge docs

Where the AI finds project patterns and conventions (used mainly by the full flow):

| Topic | Location |
|---|---|
| Backend architecture & conventions | `server/architecture.md` |
| Backend implementation flow | `server/flow.md` |
| Backend spec generation | `server/feature-generation-flow.md` |
| Backend testing patterns | `server/testing.md` |
| Backend postprocess checklist | `server/postprocess.md` |
| Frontend equivalents | `client/architecture.md`, `client/flow.md`, `client/feature-generation-flow.md`, `client/testing.md`, `client/postprocess.md` |
| Feature specs | `server/features/*.md`, `client/features/*.md` |
| Git conventions | `git-workflow.md` |

## AI tool

- **Primary tool(s) used on this project:** Cursor and/or Claude Code (both are supported; the
  adapters live in `.cursor/` and `.claude/`).
- Tool permission config: Cursor CLI → `.cursor/cli.json`; Claude Code → `.claude/settings.json`.
  Managed by `/setup-ai`.
