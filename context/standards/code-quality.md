# Code Quality (universal)

These are defaults. The **project's own** conventions override every line here —
when in doubt, match the surrounding code.

## Match the codebase
- Read 3-5 similar files before writing a new one.
- Reuse the existing import/export style, naming, and formatting. Do not
  introduce a second way to do something the repo already does.
- Prefer existing dependencies over adding new ones.

## Structure
- Small, single-purpose functions. Clear names over clever names.
- Separate concerns; keep side effects at the edges.
- Validate input at boundaries; trust internal invariants.

## Errors
- Handle errors explicitly; never swallow them silently.
- Preserve context when wrapping an error.
- Fail loudly in development, degrade gracefully in production.

## Types (when the language has them)
- Prefer explicit types at module boundaries; let inference handle locals.
- Avoid `any`/untyped escapes in new code.

## Comments
- Explain *why*, not *what*. Minimal, high-signal. No commented-out code.

## Safety
- Never commit secrets, tokens, or credentials.
- Parameterize queries; never string-concatenate untrusted input into SQL/shell.

## Definition of done
- Lint/typecheck clean for the files you touched.
- No new warnings introduced.
- Change is scoped to the request; unrelated edits are called out.
