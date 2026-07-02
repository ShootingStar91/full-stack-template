# Testing Patterns

This document describes testing patterns and conventions for the server application. See [`architecture.md`](./architecture.md) for overall architectural patterns.

## Domain Organization and Test Suites

Some features require new domains according to domain-driven development as described in [`architecture.md`](./architecture.md), and some are extensions to existing domains.

When a feature extends an existing domain, the test suite mostly or completely extends existing test suites and, if necessary, refactors or changes existing tests. New test files are only created when adding new functionality that doesn't fit into existing test files.

When a feature creates a new domain, new test files are created following the test file naming conventions.

## Test File Naming

Tests are co-located with the code they test:

- `.test.unit.ts` - Unit tests (testing functions in isolation).
- `.test.integration.ts` - Integration tests (testing services with database).
- `.test.graphql.api.ts` - GraphQL API tests (testing full GraphQL API).
- `.test.rest.api.ts` - REST API tests (testing full REST API).

## Running Tests

The test runner uses Vitest configured via the `MODE` environment variable to run specific test types. Use the Taito CLI to run tests — it takes care of the environment and containers for you.

**Prerequisites:**
- The integration and e2e (API) tests run against the running stack, so start it first: `taito start`
- Unit tests have no external dependencies and can be run without starting the stack

**Run all server tests:**
```bash
# Unit tests (no stack required)
taito test-unit:server

# Integration and e2e/API tests (requires `taito start`)
taito test:server
```

**Run a specific test:**
```bash
# A single unit test
taito test-unit:server my-test

# A specific integration/e2e test suite and test
taito test:server my-suite my-test
```

`taito test-unit:server` runs the unit tests on the host, while `taito test:server` runs the
integration and e2e tests inside the container against the running application. The same commands
work locally and in CI/CD — the CI pipeline runs `taito test` automatically.

## Test Types

**Unit Tests (`.test.unit.ts`):**
- Test pure functions and utilities in isolation
- No database or external dependencies
- Fast execution
- Use standard Vitest testing patterns

**Integration Tests (`.test.integration.ts`):**
- Test services with database access
- Use `makeTestContext()` to create test context
- Access test data via `globalThis.testData`
- Test business logic and authorization

**API Tests (`.test.graphql.api.ts` / `.test.rest.api.ts`):**
- Test full API endpoints (GraphQL or REST)
- Use test clients (`graphql`, `clientWithUser` for GraphQL)
- Test end-to-end request/response flow
- Test authentication and authorization

## Integration Tests

Integration tests use `makeTestContext()` to create a test context:

```typescript
import { makeTestContext } from '~/test/test-utils';
import { organisationService } from './organisation.service';

describe('organisation service', async () => {
  it('return organisation', async () => {
    const ctx = await makeTestContext({ user: 'admin' });
    
    const data = await organisationService.getOrganisation(
      ctx,
      globalThis.testData.organisation.id
    );
    
    expect(data).toBeDefined();
  });
});
```

## GraphQL API Tests

GraphQL API tests use the test client:

```typescript
import { graphql, clientWithUser } from '~/test/graphql-test-client';

describe('Organisation API', () => {
  it('should return all organisations', async () => {
    const data = await clientWithUser('admin').request(
      graphql(`
        query Organisations {
          organisations {
            id
            name
          }
        }
      `)
    );
    
    expect(data.organisations).toBeDefined();
  });
});
```

## Test Data

Test data is available via `globalThis.testData` which contains pre-seeded test users and organisations:

- `globalThis.testData.users` - Test users (admin, manager, viewer roles)
- `globalThis.testData.organisation` - Test organisation
- `globalThis.testDb` - Test database connection

Test data is automatically seeded before tests run. Use these pre-seeded entities rather than creating new test data.
