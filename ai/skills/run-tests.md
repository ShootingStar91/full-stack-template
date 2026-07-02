---
name: run-tests
description: Runs all tests (client and server unit tests, server integration/e2e tests, and Playwright E2E tests) using the Taito CLI. Use when the user asks to run tests, verify tests, or check test status.
disable-model-invocation: false
---

# Run Tests

Runs all test suites for the project: client and server unit tests, server integration/e2e tests,
and Playwright E2E tests.

> **Commands are project-specific.** The commands below are the full-stack-template defaults
> (Taito CLI, which handles the environment and containers). Always check the **Tests** and
> **Running the app** tables in `ai/project.md` first — if this project defines different commands
> there, use those instead of the ones written below.

## When to Use

- User asks to "run tests" or "run all tests"
- User wants to verify test status
- Before committing code changes
- After implementing features
- When debugging test failures

## Test Execution Flow

The skill runs tests in this order:

1. **Unit Tests** - Fast, no dependencies (run on the host)
   - Client unit tests
   - Server unit tests
2. **Server Integration & E2E Tests** - Require the application stack to be running
3. **Playwright E2E Tests** - Require the full application stack to be running

## Instructions

### Step 1: Run Unit Tests

Unit tests are fast and have no external dependencies. Run them first:

```bash
# Client unit tests
taito test-unit:client

# Server unit tests
taito test-unit:server
```

(You can run all unit tests across every container at once with `taito test-unit`.)

**Expected output**: Vitest test results for the unit tests

**If tests fail**: Report failures and stop execution. Do not proceed to the integration/e2e tests.

### Step 2: Automatically Ensure the Application Stack is Running

**CRITICAL**: Never ask the user to start the stack. Start it automatically if needed.

The integration, e2e, and Playwright tests run against the live application stack. Before running
them, check whether the stack is up:

```bash
taito curl server
```

**If the stack is NOT running**:

1. **Automatically start** it (do not ask user):
   ```bash
   taito start
   ```
2. Wait for the stack to be ready by polling `taito curl server` with short incremental waits.

**If the stack is already running**: Proceed to Step 3.

**Never prompt the user**: If the stack is not running, start it automatically without asking.

### Step 3: Run Server Integration & E2E Tests

Run the server integration and e2e tests against the running stack:

```bash
taito test:server
```

**Expected output**: Test results for the server integration and e2e suites

**If tests fail**: Report failures and stop execution. Do not proceed to Playwright tests.

### Step 4: Run Playwright E2E Tests

Playwright E2E tests require the full application stack (ensured in Step 2). Run them with:

```bash
taito test:playwright
```

**Expected output**: Playwright test results

**If tests fail**: Report failures

**Never prompt the user**: If the app stack is not running, start it automatically (Step 2) without asking.

## Complete Command Sequence

For local development, run:

```bash
# 1. Unit tests (fast, no dependencies)
taito test-unit

# 2. Ensure the application stack is running (start it if not)
taito curl server || taito start

# 3. Server integration & e2e tests
taito test:server

# 4. Playwright E2E tests
taito test:playwright
```

You can also run every integration/e2e and Playwright suite together with a single `taito test`
after the stack is running.

## Environment Handling

- **Unit tests** (`taito test-unit`, `taito test-unit:client`, `taito test-unit:server`) run on the
  host and need no running stack.
- **Integration, e2e, and Playwright tests** (`taito test:server`, `taito test:playwright`,
  `taito test`) run against the application stack — start it first with `taito start`.
- The same commands work locally and in CI/CD; the CI pipeline runs `taito test` automatically.

## Error Handling

- **Unit tests fail**: Stop execution, report failures
- **Stack won't start**: Report error, suggest running `taito trouble` or checking Docker
- **Integration/e2e tests fail**: Stop execution, report which suite failed
- **Playwright tests fail**: Report failures but don't block (E2E tests are often flaky)

## Output Format

After running all tests, provide a summary:

```
Test Execution Summary:
✅ Client unit tests: [X] passed, [Y] failed
✅ Server unit tests: [X] passed, [Y] failed
✅ Server integration/e2e tests: [X] passed, [Y] failed
✅ Playwright E2E tests: [X] passed, [Y] failed

Overall Status: [PASS/FAIL]
```

## Notes

- **CRITICAL**: **NEVER ask the user to start services**. Always start the stack automatically with `taito start` if needed.
- Unit tests are fast and have no external dependencies.
- Integration, e2e, and Playwright tests require the application stack — start it automatically if not running.
- When starting the stack, run it in the background (`is_background: true`) so it doesn't block test execution.
