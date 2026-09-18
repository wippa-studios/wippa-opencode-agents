# Context Navigation (global)

Global fallback used when a project has no local context (`.opencode/context/`
or `context/`). Project context always wins. Keep every file under ~150 lines.

## Standards
- `standards/code-quality.md` — universal code-quality expectations
- `standards/test-coverage.md` — what "tested" means

## Workflows
- `workflows/task-breakdown.md` — how to decompose and order work

## Project intelligence (never global)
Tech stack, naming conventions, and project-specific patterns belong in the
project's own context files. Do not guess them from here — read the repo.
