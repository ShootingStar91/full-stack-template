# Claude Code Rules

The AI development setup for this project is shared across AI tools and lives in the `ai/`
directory.

**At the start of every session:**

1. Read `ai/rules.md` and follow it — it is the session entry point.
2. Read `ai/project.md` — it defines the active development flow and all project-specific
   commands (running the app, tests, lint, database migrations).

Skills in `.claude/skills/` and agents in `.claude/agents/` are thin adapters; their real
instructions live in `ai/skills/` and `ai/agents/`.

To customize this setup for the project (commands, permissions, choice of flow), run the
`/setup-ai` skill.
