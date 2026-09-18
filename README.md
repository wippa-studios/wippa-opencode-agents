# ⚡ Wippa OpenCode Agents

A streamlined, high-efficiency multi-agent configuration for OpenCode.

Wippa OpenCode Agents eliminates agent sprawl by utilizing one unified primary agent that dynamically plans, builds, reviews, tests, and verifies code in a single autonomous flow. By intelligently scaling its approach based on task complexity, it keeps your workspace clean, your workflow fast, and your operations secure.

## ✨ Key Design Philosophy

**One Primary Agent:** Say goodbye to endless hand-offs. `plan-then-build` is the only active agent. Default OpenCode Build and Plan agents are disabled. Subagents are invoked strictly on-demand, minimizing overhead.

**Adaptive Velocity:** Why run a 6-phase pipeline for a typo? Tasks touching 3 or fewer files use a lightning-fast 3-phase flow. Complex tasks automatically trigger the full pipeline.

**Intelligent Permissions:** Built-in regex-based security. The `smart-approval` plugin acts as the single source of truth for deny rules, offering infinitely more flexibility than standard glob patterns.

**Minimal Artifacts:** No messy workspaces. Simple tasks only update `PROGRESS.md`. Complex tasks generate a concise `PLAN.md` and `REPORT.md`—keeping your repository clean of unnecessary markdown trails.

## 🚀 Installation

Install the configuration directly into your OpenCode config directory.

```bash
# 1. Clone the repository into your OpenCode configuration folder
git clone https://github.com/wippa-studios/wippa-opencode-agents.git ~/.config/opencode

# 2. Navigate to the directory
cd ~/.config/opencode

# 3. Install the required plugin dependencies
npm install
```

> **Note:** If you already have an existing OpenCode configuration, you can seamlessly integrate Wippa by copying the relevant files from this repository into your setup.

## 🔄 Adaptive Workflows

The primary agent dynamically scales its execution pipeline based on the scope of the request.

### 🏃‍♂️ Fast Path (Simple Tasks, ≤3 Files)

Designed for bug fixes, minor feature additions, and refactoring.

1. **ASSESS:** `context-scout` runs (if in a new repo) to read 3-5 similar files and align with codebase conventions. Generates a lightweight `PLAN.md`.
2. **BUILD:** Executes implementation, linting, and type-checking.
3. **VALIDATE:** Conducts self-review, runs relevant tests, and updates `PROGRESS.md`.

### 🏗️ Deep Path (Complex Tasks, >3 Files)

Designed for architectural shifts, multi-component features, and wide-scale refactoring.

1. **EXPLORE:** Deep codebase search and convention discovery.
2. **PLAN:** High-level architecture design and comprehensive `PLAN.md` generation.
3. **BUILD:** Incremental, step-by-step implementation.
4. **REVIEW:** Dual-layer review utilizing self-check capabilities plus the `code-reviewer` subagent.
5. **TEST:** Delegates to the `test-engineer` subagent for comprehensive test authoring and execution.
6. **VERIFY:** Final quality gates (ensuring tests pass, no critical security findings, and strict adherence to the initial scope).

## 📂 Repository Architecture

```
wippa-opencode-agents/
├── opencode.jsonc              # Core agent config, permissions, & primary agent routing
├── prompts/
│   └── plan-then-build.txt     # Main system prompt driving the unified workflow (132 lines)
├── plugin/
│   └── smart-approval.ts       # Adaptive permission plugin handling regex-based deny rules
├── agent/                      # On-demand Subagent definitions
│   ├── context-scout.md        # Read-only convention & style discovery
│   ├── task-manager.md         # Atomic subtask breakdown for complex (8+ file) jobs
│   ├── code-reviewer.md        # Diff review & security analysis
│   └── test-engineer.md        # Test suite authoring & execution
├── skills/
│   ├── repo-conventions/       # Logic for matching existing codebase patterns
│   └── progress-tracker/       # Multi-step PROGRESS.md protocol management
└── context/
    ├── standards/              # Baselines for code quality and test coverage
    └── workflows/              # Rule sets for task breakdown
```

## 🤖 The Subagent Ecosystem

Subagents in Wippa do not run in the background. They are highly specialized tools invoked via `task()` exclusively when the primary agent requires their specific expertise.

| Agent | Role | When it is Triggered |
|-------|------|----------------------|
| `context-scout` | Convention discovery & style matching | First task in an unfamiliar repository |
| `task-manager` | Atomic subtask breakdown & tracking | Complex, multi-component workflows (8+ files) |
| `code-reviewer` | Diff review & static security analysis | Non-trivial architectural changes |
| `test-engineer` | Test authoring & execution | Post-implementation validation phase |

## 🛡️ Security: Smart Approval Plugin

Wippa replaces static, rigid configuration files with the `smart-approval` TypeScript plugin. This intercepts all permission decisions, auto-approving routine development work while rigidly blocking destructive or out-of-scope operations.

**Hard-Blocked Actions:**

- **Destructive Bash:** `sudo`, `rm -rf /`, `mkfs`, `dd`, `shutdown`, fork bombs, `curl | sh`
- **Sensitive Edits:** `.env`, `.key`, `.pem`, `.ssh/`, `.git/`, `node_modules/`
- **External Directories:** Access outside the workspace, including `~/.ssh/`, `~/.aws/`, `~/.gnupg/`, `~/.kube/`

Everything else is safely auto-approved to maintain velocity. The plugin operates on a strict "deny-first for sensitive paths" model and will never weaken a structurally configured deny rule.

## 🧠 Autonomy & Interaction Rules

The Wippa primary agent is designed to operate seamlessly with minimal human intervention. It follows a strict behavioral protocol:

**Act by Default:** State the plan, execute the plan. No waiting for routine approvals.

**Ask Only When Necessary:** The agent will only pause for human input if a request is genuinely ambiguous (and a wrong guess would waste work) or if an action is irreversible/outside the workspace.

**Strictly Prohibited Behaviors:**

- Asking the same question twice.
- Asking "should I proceed?" after stating a clear plan.
- Committing, pushing, or opening PRs unless explicitly instructed by the user.

## 📄 License

This project is licensed under the MIT License.
