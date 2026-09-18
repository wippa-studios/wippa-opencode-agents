# Test Coverage (universal)

The project's testing conventions override these defaults — read existing tests
first and match them.

## What "tested" means
- Every new behavior has at least one **positive** and one **negative** test.
- Edge cases: empty, null/undefined, boundary values, error paths, concurrency
  where relevant.
- A test that cannot fail is not a test.

## Structure
- Arrange-Act-Assert, one behavior per test.
- Descriptive names: `<unit> <condition> <expected outcome>`.

## Determinism
- Mock network, clock, randomness, and third-party services.
- No sleep-based timing; no reliance on execution order.
- Tests must pass reliably in a clean environment.

## Running
- Run the suite before declaring work done; report the exact command.
- Never delete or weaken a failing test to make the suite green — fix the code or
  report the failure.
