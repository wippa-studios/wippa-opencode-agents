# Wippa OpenCode Agents

A streamlined multi-agent configuration for [OpenCode](https://opencode.ai) — one unified agent that plans, builds, reviews, tests, and verifies code in a single autonomous flow.

## What's Inside

```
opencode.jsonc              # Agent config — single primary agent, permissions
prompts/plan-then-build.txt # The main system prompt (132 lines)
plugin/smart-approval.ts    # Adaptive permission plugin (regex-based deny rules)
agent/                      # Subagent definitions
  context-scout.md          # Convention discovery (read-only)
  task-manager.md           # Atomic subtask breakdown (8+ files)
  code-reviewer.md          # Diff review + security analysis
  test-engineer.md          # Test authoring + execution
skills/
  repo-conventions/         # Match existing codebase patterns
  progress-tracker/         # PROGRESS.md protocol for multi-step work
context/
  standards/                # Code quality + test coverage defaults
  workflows/                # Task breakdown workflow
```

## Key Design Decisions

**One agent, not five.** `plan-then-build` is the only active agent. Build and Plan stock agents are disabled. Subagents (`context-scout`, `code-reviewer`, etc.) are invoked on-demand via `task()` — they don't run unless needed.

**Fast path for simple work.** Tasks touching ≤3 files use a 3-phase flow: `ASSESS → BUILD → VALIDATE`. The full 6-phase pipeline only kicks in for multi-component work.

**Permissions via plugin, not config.** The `smart-approval` plugin is the single source of truth for deny rules. The JSON config only declares broad allows for read-only tools. This means deny rules are regex-based and more flexible than glob patterns.

**Minimal artifacts.** Simple tasks update `PROGRESS.md` only. Complex tasks produce `PLAN.md` + `REPORT.md` — not 5 separate files per task.

## Install

Copy this directory to `~/.config/opencode/`:

```bash
git clone https://github.com/wippa-studios/wippa-opencode-agents.git ~/.config/opencode
cd ~/.config/opencode
npm install  # installs the plugin dependency
```

Or add it to an existing config by copying the relevant files.

## Workflow

### Simple tasks (≤3 files)

```
ASSESS → context-scout (if new repo) → read 3-5 similar files → PLAN.md
BUILD   → implement, lint, typecheck
VALIDATE → self-review, run tests, update PROGRESS.md
```

### Complex tasks (>3 files)

```
EXPLORE → convention discovery, codebase search
PLAN    → architecture design, PLAN.md
BUILD   → implement incrementally
REVIEW  → self-review + code-reviewer subagent
TEST    → test-engineer subagent or direct
VERIFY  → self-check gates (tests pass, no critical findings, scope matches)
```

## Autonomy Rules

- **Act by default.** State the plan, execute it. No routine approvals.
- **Ask only when:** genuinely ambiguous + wrong guess wastes work, or irreversible action outside workspace.
- **Never:** ask twice, ask "should I proceed?", commit/push/PR unless explicitly asked.

## Subagents

| Agent | Role | When to use |
|-------|------|-------------|
| `context-scout` | Convention discovery | First task in unfamiliar repo |
| `task-manager` | Subtask breakdown | 8+ files, multi-component |
| `code-reviewer` | Diff review + security | Non-trivial changes |
| `test-engineer` | Test authoring + execution | After implementation |

## Plugin: Smart Approval

The `smart-approval` plugin intercepts permission decisions and auto-approves routine work while blocking dangerous operations:

- **Bash**: blocks `sudo`, `rm -rf /`, `mkfs`, `dd`, `shutdown`, fork bombs, `curl|sh`
- **Edit**: blocks `.env`, `.key`, `.pem`, `.ssh/`, `.git/`, `node_modules/`
- **External dirs**: blocks `~/.ssh/`, `~/.aws/`, `~/.gnupg/`, `~/.kube/`

Everything else is auto-approved. The plugin never weakens a configured deny.

## License

MIT
