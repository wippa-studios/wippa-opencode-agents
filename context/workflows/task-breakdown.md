# Task Breakdown (universal)

Use for work spanning 4+ files or multiple components. For 1-3 files, execute
directly.

## Steps
1. Read the SPEC/FINDINGS.
2. Identify functional components and their boundaries.
3. Split into atomic subtasks (each independently verifiable, ~<2h agent work).
4. Map dependencies (`depends_on`); mark independent tasks `parallel: true`.
5. Define binary acceptance criteria and concrete deliverables per subtask.

## Ordering rules
- Foundations first (schema, types, contracts), then features, then integration.
- Define interfaces/contracts before implementing both sides.
- Never parallelize subtasks that touch the same file.
- Sequence: independent batch → dependents → integration/validation.

## Where it goes
Write task JSON under `.tmp/agent/{slug}/tasks/` (see the `task-manager`
subagent for the exact schema). The dependency graph is the execution plan:
run batches in order, tasks within a batch together.

## Anti-patterns
- Subtasks that are really the whole feature.
- Hidden dependencies (same file, shared resource) not declared.
- Acceptance criteria that are not pass/fail.
