# AI Rules — Session Entry Point

These rules apply to every AI session in this project, regardless of tool (Cursor, Claude Code,
etc.) and regardless of flow. Tool-specific entry files (`.cursorrules`, `CLAUDE.md`) point here.

## 1. Read the project configuration

At the start of a session, read **`ai/project.md`**. It defines:

- the **active flow** (how development tasks are carried out),
- the **commands** for running the app, tests, lint, and database migrations,
- the **migrations policy** (whether the AI may run migrations itself),
- where the **knowledge docs** (architecture, testing, flows) live.

`ai/project.md` always wins over commands or paths mentioned anywhere else in the AI docs.

If `ai/project.md` is still marked as containing **template defaults** and something doesn't match
this project (e.g. a command fails because the project doesn't use the Taito CLI), mention that the
user can run the `/setup-ai` skill to customize the setup — then continue as best you can.

## 2. Follow the active flow

For development tasks (edits, features, fixes), read and follow the flow file named as **Active
Flow** in `ai/project.md`:

- `ai/flows/small-flow.md` — implement → test → lint → commit. Lightweight default for small work.
- `ai/flows/full-flow.md` — specs-driven development with validation gates and verifier subagents.

The user can override the flow for a single task by asking explicitly (e.g. *"use the full flow
for this one"*). Requests that clearly use full-flow triggers (*"let's generate a feature spec"*)
follow the full flow even if small-flow is active.

Questions, explanations, and reviews that don't change code need no flow.

## 3. Running the app

- "Running the app" means the full stack (client, server, database) in dev mode, using the
  commands in `ai/project.md`.
- **Never ask the user to start the app.** If a task needs the app running (tests, layout
  iteration, live checks, debugging), check the health command from `ai/project.md`; if the stack
  is down, start it automatically in the background and poll the health check with short waits
  until it responds.
- If startup fails, report the error with troubleshooting suggestions — still don't ask the user
  to start it manually.

## 4. Git

- Follow the conventions in `git-workflow.md` (branching, Angular/commitlint commit format); use
  the `/git-workflow` skill for git operations.
- Before starting implementation work: `git status` — if on `dev` or `master`, branch from `dev`
  first (`git checkout -b feature/<name> dev`). Never commit new work directly to `dev`/`master`.
- Before committing: lint/typecheck must pass (command in `ai/project.md`); running unit tests is
  recommended.
- **Push only when the user asks.**

## 5. General

- Ask for clarification when a task is ambiguous in a way that changes the outcome.
- Keep changes scoped to the task; follow existing code conventions.
- To change how any of this works, the user can edit the files under `ai/` directly or ask the AI
  to edit them — or re-run `/setup-ai`.
