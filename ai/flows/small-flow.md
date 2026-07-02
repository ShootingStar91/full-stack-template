# Small Flow

A lightweight development flow for small edits and small features. No feature specs, no session
boundaries, no validation gates — just: implement, test, lint, commit.

Use this flow when it is the active flow in `ai/project.md`, or when the user explicitly asks for
it. Typical triggers: the user requests a small edit to existing functionality, a bug fix, or a
new small feature.

All commands referenced below (tests, lint, app start, migrations) come from `ai/project.md`.

## Steps

### 1. Branch check

Run `git status`. If on `dev` or `master`, create a feature branch first
(`git checkout -b feature/<kebab-case-name> dev` — see `git-workflow.md`). Otherwise stay on the
current branch.

### 2. Implement

- Read the relevant existing code first and follow its conventions (naming, structure, error
  handling). Architecture docs listed in `ai/project.md` are available if deeper context is needed.
- Keep the change minimal and scoped to what the user asked. No drive-by refactoring.
- If the change needs a database migration, follow the migrations policy in `ai/project.md`
  (create and apply it yourself only if the policy allows).
- If the request is ambiguous in a way that changes the implementation, ask before coding.

### 3. Test

- Run the tests relevant to the change using the test commands in `ai/project.md` (at minimum the
  unit tests; run integration/E2E suites when the change touches behavior they cover — start the
  app automatically if they need it).
- **Fix any failures** caused by the change.
- **Add or update tests** when the change alters behavior: new functionality gets a test, changed
  behavior gets its test updated. Follow the project's testing patterns (see the testing docs in
  `ai/project.md`). A pure refactor or copy change may not need new tests — use judgment.

### 4. Lint

Run the lint/typecheck command from `ai/project.md` and fix all reported errors.

### 5. Commit

- Stage and commit using the commit conventions in `git-workflow.md`
  (`<type>(<scope>): <subject>`).
- **Do not push** unless the user asks. When asked, use the `git-workflow` skill.

### 6. Report

Summarize briefly: what changed, which tests ran (pass/fail), what was committed. Then stop and
wait for the next instruction.

## What this flow does NOT do

- No feature spec files are written or required.
- No test plans, validation skills, or verifier subagents (unless the user asks for them).
- No multi-session task boundaries — a conversation can contain several small tasks.

If a request turns out to be large (new domain concept, several days of work, cross-cutting
change), say so and suggest the full flow (`ai/flows/full-flow.md`) instead of silently doing a
big change in small-flow mode.
