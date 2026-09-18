---
name: progress-tracker
description: Update PROGRESS.md after each meaningful step so long-running agents remember context across compactions and restarts.
origin: Custom
---

# Progress Tracker Skill

A dead-simple convention that fixes "agent forgot what it was doing" failures during multi-step tasks. Uses a `PROGRESS.md` file in the workspace root that the agent updates after each meaningful step.

## How It Works

`PROGRESS.md` is a lightweight scratchpad with three sections. Entries are **prefixed by agent name** so multiple agents can append without overwriting each other:

```markdown
# Progress: <Task Name>

### [coder] ✅ Done
- [2026-07-31 14:30] Installed multer and sharp packages
- [2026-07-31 14:15] Created avatar upload route handler with 5MB limit

### [coder] ⏭️ Next
- Add image resizing middleware (200x200, 400x400 variants)
- Write integration tests for upload endpoint

### [coder] 🧠 Decisions & Gotchas
- Decision: Store files on S3, not local disk (stateless deployment)
- Decision: sharp for resizing (fast, streaming-capable, widely used)
- Gotcha: multer's `diskStorage` doesn't work on serverless — use `memoryStorage` and stream to S3
```

**Append-only, never overwrite.** Each agent that needs to log reads the file, appends its new entries under its own `### [agent-name]` heading, and writes back. No agent edits or removes another agent's entries.

## When to Activate

- Multi-step coding tasks (implement feature → test → fix → PR)
- Any task where context might get compacted or a session might restart
- When the user says "don't lose track" or "keep notes"
- Chains of subagents (explorer → planner → coder)

## Workflow

### 1. Read — always, before every action

**Before doing anything, re-read `PROGRESS.md` from disk.** Do not rely on in-memory context of what it said. This is the single most important rule — without it, the file drifts out of sync and becomes noise.

Read it:
- At the start of a turn
- After any interruption or compaction notification
- After returning from a subagent or parallel task
- **Any time you are about to write to it** (read first, append, write back — never write blind)

### 2. Update — after each meaningful step

Append new entries under your agent's heading in each section. Keep it brief — a sentence per bullet is enough.

Use this format for the prefix:

```
### [agent-name]
```

Where `agent-name` is the agent's identifier from opencode.json: `orchestrator`, `coder`, `explorer`, `planner`, `tester`, `debug-specialist`, `devops`, etc.

Example entry block:

```markdown
### [coder]
## ✅ Done
- [2026-07-31] Set up database schema for user profiles
- [2026-07-31] Wrote migration script for existing users

## ⏭️ Next
- Add GraphQL resolver for `updateProfile` mutation
- Write unit tests for profile service

## 🧠 Decisions & Gotchas
- Decision: Using Drizzle over Prisma to avoid managed-runtime lock-in
- Gotcha: UUID primary keys need explicit `gen_random_uuid()` default in PostgreSQL 15
```

### 3. Reference — before acting

When deciding what to do next or how to do it, check the **`## ⏭️ Next`** and **`## 🧠 Decisions & Gotchas`** sections first. Don't assume the information is still loaded in context.

### 4. Cleanup — owned by the orchestrator only

The **orchestrator** owns deletion. When the entire multi-step task is complete and committed:
- Orchestrator removes `PROGRESS.md`
- Alternatively, orchestrator renames it to `PROGRESS-{task-name}-archive.md` for records

Subagents **never** delete `PROGRESS.md`. They may signal completion in their entry, but only the orchestrator or the user removes the file.

Removing the file signals "this task is done — fresh start." Leaving it in place signals that state is still active.

## Agent Prompt Snippet

In any agent that does multi-step work (applied via the `prompt` field in opencode.json):

```
PROGRESS.md protocol:
  - Read PROGRESS.md from disk BEFORE every action (not from memory).
  - Append entries under your agent heading after each meaningful step.
  - Never overwrite or delete entries from other agents.
  - Before continuing after compaction or interruption, re-read from disk.
```

## Staleness Prevention — Why This Matters

The biggest failure mode is not "agents forget to write" — it's "agents wrote earlier and rely on in-memory context instead of re-reading from disk." If you re-read before every action, the file stays authoritative. If you don't, it drifts out of sync and becomes noise. **Read from disk, always.**

## Multi-Agent Coordination

When the orchestrator chains subagents (e.g. explorer → planner → coder):
- All agents share one `PROGRESS.md`
- Each appends under its own `### [agent-name]` prefix
- No parallel writes happen (chaining is sequential)
- The orchestrator reads PROGRESS.md between subagents to decide what to dispatch next

If two agents *could* run in parallel (future scenario), they both append — there is no overwrite risk because the convention is append-only.

## Example: Full PROGRESS.md Lifecycle

```markdown
# Progress: Add user avatar upload feature

### [coder]
## ✅ Done
- [2026-07-31 14:30] Installed multer and sharp packages
- [2026-07-31 14:15] Created avatar upload route handler with 5MB limit

## ⏭️ Next
- Add image resizing middleware (200x200, 400x400 variants)
- Write integration tests for upload endpoint

## 🧠 Decisions & Gotchas
- Decision: Store files on S3, not local disk (stateless deployment)
- Decision: sharp for resizing (fast, streaming-capable, widely used)
- Gotcha: multer's `diskStorage` doesn't work on serverless — use `memoryStorage` and stream to S3

### [tester]
## ✅ Done
- [2026-07-31 15:00] Wrote integration tests for upload + resize pipeline
- [2026-07-31 14:50] All 12 tests passing

## ⏭️ Next
- None — task complete

## 🧠 Decisions & Gotchas
- Gotcha: sharp resize in Buffer mode needs `toBuffer()` not `toFile()` for testing
```