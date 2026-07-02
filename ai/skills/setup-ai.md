---
name: setup-ai
description: Interactive, re-runnable setup of the AI development environment for this project — tool permissions, project commands (app, tests, lint, migrations), and choice of development flow. Run after copying the ai/ setup into a project, or anytime to review/change the configuration.
disable-model-invocation: false
---

# Setup AI Environment

Interactively configures the AI development setup for this project. It is **safe to run any number
of times**: every step first reports the current state and asks whether the user wants to keep or
change it. Nothing is overwritten without the user seeing what will change.

The steps are: **0) permissions → 1) project commands & migrations → 2) flow usage → commit →
final summary**. Work through them in order, conversationally, one step at a time. Wait for the user's
answer at each decision point.

## Step 0: Tool Permissions

Goal: the AI tool can do routine safe operations (read/edit files in this repo, safe git commands,
the project's test/lint commands) without prompting the user every time.

1. **Determine which AI tool(s) the user uses** (Cursor, Claude Code, or both). If not obvious
   from which tool is currently running, ask.

2. **Read the existing permission config** for each tool in use:
   - Claude Code: `.claude/settings.json` (project-level, committed) — `permissions.allow` /
     `permissions.deny` arrays.
   - Cursor CLI: `.cursor/cli.json` — `permissions.allow` / `permissions.deny` arrays.
   - Cursor IDE: command allowlists live in the app settings, not in a repo file — if the user
     uses the Cursor IDE agent, tell them which commands to allowlist there and let them do it;
     don't try to write a file for it.

   **If permissions are already configured**: list them to the user, say they look already set up,
   and ask whether they want to edit them. If not, move on to Step 1.

3. **Propose this permission set** (adjust commands to match `ai/project.md`; for Claude Code use
   `Bash(...)` rules, for Cursor CLI use `Shell(...)` rules):
   - Safe git commands: `git status`, `git diff`, `git log`, `git branch`, `git checkout`,
     `git add`, `git commit`
   - Editing files inside this repository — this must be a concrete entry in the config, not
     just implied: for Claude Code add `Edit(**)` (covers the Edit, Write, and NotebookEdit
     tools for paths inside the project), for Cursor CLI add `Write(**)`
   - The project's test, lint, and app commands from `ai/project.md` (e.g. `taito test-unit`,
     `taito lint`, `taito start`, `taito curl`)
   - **Not included** (always prompt): `git push`, deleting files outside normal edits, anything
     touching remote/production environments.

4. **Show the user the exact permission entries you intend to write and ask if this is OK.**
   The full list of entries must be visible in the same message as the question — never ask
   "is this permission set OK?" while the list only exists in your reasoning or an earlier tool
   result. If you use a structured question prompt, print the list as text immediately before it.
   Apply their edits (add/remove entries), then write the config file, merging with any existing
   entries rather than replacing them. Verify the JSON is valid after writing.

## Step 1: Project Commands & Migrations

Goal: `ai/project.md` accurately describes THIS project. It is the single source of truth that all
flows and skills read.

1. **Read `ai/project.md`** and present its current values as a compact summary: how the app is
   started, how each test suite is run, how lint runs, how migrations work, and the migrations
   autonomy policy. Note whether the file is still marked as "template defaults".

2. **Ask the user what differs in their project.** Go through these, offering the current value as
   the default (fine to ask as one grouped question, then follow up on what they want to change):
   - How is the app/stack started for development? How do you check it's up? How is it stopped?
   - How are unit tests run? Integration/API tests? E2E tests? Which need the app running?
   - How are lint and typecheck run?
   - **Database migrations**: before asking anything, state the current configuration in plain
     text, e.g. "The project is currently configured to create migrations with `<command>`, apply
     them with `<command>`, and the AI is currently allowed/not allowed to create and apply
     migrations itself." Only then ask: does that match this project, and **should the AI create
     and apply migrations itself, or leave them to the user?** (Record the answer as the
     migrations policy.) Never ask whether to "keep migrations the same" without first spelling
     out what "the same" currently means.
   - Anything else unusual (special services, monorepo quirks, a package manager wrapper)?

3. **Verify where cheap**: with the user's consent, run the lint or unit-test command they gave to
   confirm it works. Don't run anything expensive or stateful without asking.

4. **Write the answers into `ai/project.md`** (update the tables and the migrations section, and
   remove/adjust the "template defaults" status note once customized).

5. **Propagate**: search the other AI files (`ai/rules.md`, `ai/flows/*.md`, `ai/skills/*.md`,
   `ai/agents/*.md`, `git-workflow.md`, and the knowledge docs listed in `ai/project.md`) for
   hardcoded commands that now contradict `ai/project.md` (e.g. `taito ...` when the project
   doesn't use Taito) and update them. Show the user a short list of the files you changed.
   Also update the Step 0 permission entries if the commands changed.

## Step 2: Flow Usage

Goal: the right **Flow Usage** mode for this team and project is set in `ai/project.md`.

1. **Report the current flow usage mode** from `ai/project.md`.

2. **Describe the available flows briefly** (read `ai/flows/` for the current list):
   - **small-flow** — the AI implements the requested small edit or feature, runs and fixes the
     relevant tests (adding tests when behavior changes), lints, and commits. No specs, no
     validation gates.
   - **full-flow** — specs-driven development: feature specs as single source of truth, one task
     per session, TDD on the backend, layout-first on the frontend, mandatory validation gates
     with verifier subagents.

3. **Ask whether flows should be used by default, or only when explicitly asked.** The brief
   flow descriptions from the previous point must appear as text immediately before this
   question, in the same message — the user needs to know what small-flow and full-flow do
   before choosing how they are triggered. Options:
   - **`default`** — the AI uses flows for every development task and infers which flow fits the
     prompt: small-flow unless the prompt requests a non-trivial new feature that likely needs
     more planning and iteration — then the full flow applies, but the AI always asks the user
     for permission before starting it.
   - **`on-request`** — the AI works normally without flows and follows a flow only when the user
     explicitly asks for one.

   Also ask whether the flows' defaults are OK as described or they'd like to change something
   (e.g. "small flow should also run E2E tests", "don't commit automatically, only stage",
   "full flow: skip the layout iteration step").

4. **Apply**: set **Flow Usage** in `ai/project.md` (this is what the session rules in
   `ai/rules.md` read on every prompt); apply any requested customizations by editing the flow
   file(s) in `ai/flows/` directly. Summarize the edits made.

## Commit

After all steps are complete, commit the configuration changes directly (no need to ask): stage
the files this setup touched (e.g. `.claude/settings.json`, `.cursor/cli.json`, `ai/project.md`,
edited flow/skill files) and commit with a message like `chore(ai): configure AI development
setup`. Do not stage unrelated pending changes, and do not push.

## Final Summary

End with a short report to the user:

- What was configured in each step (permissions written where, commands recorded, flow usage
  mode).
- **Where the flows are defined**: `ai/flows/small-flow.md` and `ai/flows/full-flow.md`, with
  project facts in `ai/project.md` and session rules in `ai/rules.md`.
- **Remind the user**: to edit, add, or omit any step of a flow, they can either edit those files
  directly or simply ask the AI to make the edit. Re-running `/setup-ai` at any time reviews the
  whole configuration again.
