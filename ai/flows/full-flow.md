# Full Flow (Specs-Driven Development)

The heavyweight, specs-driven development flow. A feature description (spec) is the single source
of truth: nothing is implemented until a spec exists, and everything implemented is validated back
against the spec by skills and independent verifier subagents.

Use this flow when it is the active flow in `ai/project.md`, or when the user explicitly asks for
it (e.g. *"let's generate a client feature"*, *"let's implement feature X"*).

All commands referenced by this flow (tests, lint, app start, migrations) come from
`ai/project.md`.

## Single Task Per Session

**CRITICAL**: In this flow, work on a **single task** per agent session.

Tasks are categorized by two axes:

1. **Task type**: Feature generation (writing a spec) OR Implementation
2. **Target**: Backend (server) OR Frontend (client)

Rules:

- **One task per session** — complete one task before starting another.
- **Never mix task types** — do not generate a spec and then implement it in the same session.
- **Never mix targets** — do not work on both backend and frontend in the same session.
- **Session ends when the workflow completes** — wait for the user's next instruction.

When starting a session, identify the task type, the target, and (for implementation) the feature
name, e.g. *"Implement frontend feature: post-list (from client/features/post-list.md)"*.

## Task Boundaries

- **Feature generation** creates or updates spec files only:
  - Backend: `server/features/*.md` — Frontend: `client/features/*.md`
- **Implementation** generates code, tests, and related files based on a spec:
  - Backend: `server/src/`, tests, migrations, etc. — Frontend: `client/src/`, tests, routes,
    components, etc.
- Test plans and summaries are plain text output — never saved as `.md` files. The spec is the
  only markdown a workflow writes.

## Workflow A — Feature Generation

1. Identify target (backend or frontend).
2. Follow `server/feature-generation-flow.md` or `client/feature-generation-flow.md`.
3. Generate sections one by one, getting user confirmation after each.
4. **Before saving the spec** (mandatory, never skip, never ask permission):
   - Run the `/validate-feature-spec` skill on the completed spec.
   - Launch the `spec-validator` subagent for independent deep validation.
   - Address all issues before saving.
5. Save the spec in the appropriate `features/` folder. **Session ends.**

## Workflow B — Implementation

1. **Check git branch**: run `git status` — if on `dev` or `master`, create a feature branch from
   `dev` first (see `git-workflow.md`).
2. Read the feature spec from the `features/` folder.
3. Follow the appropriate flow document:
   - Backend: `server/flow.md` (test-driven: test plan → tests → code)
   - Frontend: `client/flow.md` (layout-first: layout → Figma match → interactions → tests)
4. Reference the architecture docs (`server/architecture.md` / `client/architecture.md`) for
   patterns, and the postprocess checklists for final quality review.
5. **Session ends** when the flow's summary step is done — do not start new features.

## Mandatory Validation Gates

These run automatically at fixed points. **Never skip them, never make them conditional, never ask
permission to run them.** Report results to the user; fix issues before advancing past a gate.

| Gate | When | What runs |
|---|---|---|
| 1 | After spec generation, before saving | `/validate-feature-spec` skill + `spec-validator` subagent |
| 2 | After test plan (backend step 1 / frontend step 5) | `test-plan-validator` subagent |
| 3 | After test generation (backend step 2 / frontend step 6) | `/check-test-coverage` skill |
| 4 | After implementation (step 4) | `/verify-implementation` skill + `implementation-verifier` subagent + `/check-architecture-compliance` skill |

**Error handling**: if a validation skill or subagent fails to run, report the error, attempt
manual validation, and ask the user for guidance if a critical validation cannot run.

## Other Rules

- **Spec updates during implementation**: specs may be updated if security, scalability, or
  performance concerns are found (see the flow files for details).
- **Running the app**: start it automatically whenever needed, using the commands in
  `ai/project.md` — never ask the user to start it.
- **Git**: follow `git-workflow.md`; use the `/git-workflow` skill for git operations. Push only
  when the user asks.
