---
name: repo-conventions
description: Read the actual repo structure, config, and existing patterns before writing code.
origin: Custom
---

# Repo Conventions Skill

**Before writing any new code, read:**
1. Existing files in the same directory
2. Config files (tsconfig, eslint, pyproject.toml, etc.)
3. Other files of the same type to match style

## Workflow

### 1. Find config files
Check for: `tsconfig.json`, `eslint.config.*`, `prettier.config.*`,
`pyproject.toml`, `ruff.toml`, `go.mod`, `Cargo.toml`, `package.json`,
`.editorconfig`

### 2. Read 3-5 similar files
Match the kind you're writing — component, route, test, migration.

### 3. Match these patterns exactly
- Import style (relative vs absolute, barrel files)
- Export style (default vs named)
- Naming conventions (PascalCase, camelCase, kebab-case)
- Error handling (try/catch, Result types, exceptions)
- Async patterns (async/await, promises)
- Type annotations (explicit vs inferred)

### 4. Reuse existing deps
Don't introduce new libraries if existing ones solve it.

## When to Activate
- First task in a repo (always)
- New directory/module
- Before writing tests (match test patterns)
