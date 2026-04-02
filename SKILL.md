---
name: agent-handoff
description: |
  Generate a structured cross-session context handoff summary for seamless
  continuation in a new Claude Code session. Use when: context is getting long,
  quality is degrading, you want to start fresh while preserving context, or
  any phrase like "handoff", "交班", "上下文交接", "session handoff", "context summary".
  Produces a portable plain-text document optimized for a new agent to consume.
  Do NOT use for: general status reports, debugging, or code generation tasks.
argument-hint: "[--brief]"
allowed-tools:
  - Bash
  - Read
  - Write
  - Edit
  - Grep
  - Glob
---

# Agent Handoff Skill

You are generating a cross-session context handoff summary.

## Purpose

Use /handoff when the session context is getting too long, quality is degrading, or you want to start fresh while preserving essential context. This generates a structured context summary that a NEW AGENT can consume to continue work seamlessly.

CRITICAL: The output is written FOR A NEW AGENT, not for the user. Every sentence should be actionable instructions or concrete state that another agent instance can directly use. Avoid human-readable summaries, status reports, or explanatory prose.

Arguments: $ARGUMENTS

---

## GIT CONTEXT (pre-collected)

- Current branch: !`git branch --show-current 2>/dev/null || echo "(detached or no branch)"`
- Recent commits:
!`git log --oneline -15 2>/dev/null || echo "(no commits yet)"`
- Recent file changes:
!`git diff --stat HEAD~10..HEAD 2>/dev/null || echo "(no recent commits)"`
- Uncommitted changes:
!`git status --short 2>/dev/null || echo "(not a git repo)"`
- Current commit SHA: !`git rev-parse --short HEAD 2>/dev/null || echo "N/A"`

---

## Execution Flow

### PHASE 1: Validate Request

Before proceeding, confirm:
- There is meaningful work or context in this session worth preserving
- If the session is nearly empty or has no meaningful context, inform the user there is nothing to hand off, then stop
- If all tasks are complete and there is nothing pending, warn the user that a handoff may not be necessary, but proceed if they confirm

### PHASE 2: Gather Deep Context

On top of the pre-collected git context, perform additional collection:

1. **Task status**: Call TaskList or read TODO files for current task progress (if any active tasks exist)
2. **Project instructions**: Check if `CLAUDE.md` or `.claude/CLAUDE.md` exists in the project, read key instructions if found
3. **Session context review**: Review the entire conversation history and extract:
   - User's original requests (verbatim)
   - Work that was completed
   - Technical decisions made and their rationale
   - Constraints, limitations, and caveats discovered
   - Key files discussed or modified

### PHASE 3: Extract and Structure

IMPORTANT: You are writing instructions for a new agent, not a report for the user.

Write everything as direct, actionable context:
- BAD: "The project uses a hybrid approach combining lightweight and full protocol features"
- GOOD: "When adding new features, only auto-detect CLAUDE.md and TaskList. Do not add SCOPE.yml or STATUS.yml dependencies."

- BAD: "The user prefers English for all command content"
- GOOD: "All content in handoff.md must be in English. Do not mix Chinese and English."

- BAD: "Future work may include testing the handoff output quality"
- GOOD: "Next task: test handoff output by running /handoff in a session with active work, verify the output is agent-consumable"

Extraction focus:
- Convert decisions into rules the new agent must follow
- Convert completed work into current state the new agent can verify
- Convert constraints into direct imperatives
- Only include files the new agent can actually access (not gitignored or external references)
- USER REQUESTS must be quoted verbatim, never paraphrased
- If user requests reference temporary or session-scoped files (e.g. ~/.claude/plans/, /tmp/, session-generated files), inline the key content into TASK or PENDING — the new agent will NOT have access to those files
- If there are no pending tasks, state what was last completed and what logical next steps would be
- Review your own mistakes during this session — wrong approaches tried, dead ends hit, incorrect assumptions made — and record them in GOTCHAS so the new agent does not repeat them
- Preserve the last 3-5 concrete operations (exact commands, API calls, file edits) as LAST OPERATIONS — this gives the new agent few-shot examples of how things are done in this project, preventing it from guessing at operational patterns

Guiding questions for extraction:
- What rules must the new agent follow when modifying this codebase?
- What is the current state the new agent will find when it reads the files?
- What task should the new agent work on next?
- What mistakes did I make that the new agent should avoid?
- What user preferences must be respected?

### PHASE 4: Format Output

Check if `$ARGUMENTS` contains `--brief`.

**If `--brief` mode**, output the condensed version only:

```
HANDOFF CONTEXT (BRIEF)
=======================

TASK
----
[Concrete next action for the new agent. If no pending task, state "No pending task. Last completed: X"]

STATE
-----
- Branch: [branch name] @ [commit SHA]
- [Build/test status if applicable]
- [Key state the new agent needs to know]

PENDING
-------
- [Concrete tasks with enough detail to execute]
- [Blockers with specific error messages or file paths]

FILES
-----
- [path/to/file1] - [what to do with it or why it matters]
- [path/to/file2] - [what to do with it or why it matters]
(Maximum 10 files, only files the new agent can access)
```

**If full mode (default)**, output the complete version:

```
HANDOFF CONTEXT
===============

USER REQUESTS (AS-IS)
---------------------
- [Exact verbatim user requests - NOT paraphrased]

TASK
----
[Concrete next action for the new agent. If no pending task, state "No pending task. Last completed: X"]

COMPLETED
---------
- [What was done, stated as verifiable facts: "file X now contains Y", "endpoint Z returns W"]
- [Include file paths so the new agent can verify]

LAST OPERATIONS (few-shot for new agent)
-----------------------------------------
- [Last 3-5 concrete operations with exact commands/steps, so the new agent can see HOW things were done in this project]
- Example format:
  - "Ran: pnpm build -> output: 3 packages built, 0 errors"
  - "Tested: curl http://localhost:3000/api/healthz -> {status: ok}"
  - "Edited: apps/api/src/routes/auth.ts:42 — changed bcrypt rounds from 10 to 12"
- This serves as few-shot examples for the new agent to follow the same operational patterns

STATE
-----
- Branch: [branch name] @ [commit SHA]
- Uncommitted changes: [yes/no, what files]
- [Build/test status if applicable]
- [Environment or configuration state that affects work]

PENDING
-------
- [Concrete tasks with enough detail to execute without asking the user]
- [Blockers with specific error messages or file paths]
- [Current task status from TaskList if available]

FILES
-----
- [path/to/file1] - [what to do with it or why it matters]
- [path/to/file2] - [what to do with it or why it matters]
(Maximum 10 files, only files the new agent can access)

RULES
-----
- [Direct imperatives the new agent must follow, derived from decisions and constraints]
- [User preferences stated as rules: "Always do X", "Never do Y"]
- [Codebase conventions: "Commit messages use conventional commits", "Tests go in __tests__/"]
- If none, write: None

GOTCHAS
-------
- [Specific technical pitfalls: "git log fails on empty repo without 2>/dev/null fallback"]
- [Non-obvious behaviors: "plugin install changes command name to /agent-handoff:handoff"]
- [Things that look wrong but are intentional]
- [Mistakes I made during this session — format as: "Tried X, failed because Y, fix was Z"]
  Examples:
  - "Tried using @hono/node-ws v1.2 upgradeWebSocket(ws => ...), got type errors. v1.3 changed signature to (c) => ({ onOpen(evt, ws) }). Use v1.3 signature."
  - "Initially put skills/ directory for plugin structure, but this project is a command not a skill. Restructured to commands/ directory."
  - "Ran git push to main branch, but project uses master. Had to delete remote main and re-push to master."
```

### Output Rules

- Plain text with bullets format
- Use `===` and `---` underline style for headers, no `#` markdown headers
- No bold, italic, or code fences within content
- Use workspace-relative paths for files
- Every bullet must be actionable or verifiable by the new agent
- Do not include files the new agent cannot access (gitignored dirs, external references, temp files)
- USER REQUESTS (AS-IS) must contain verbatim quotes only

---

## Post-Output Instructions

After the summary, append these usage instructions:

```
---

TO CONTINUE IN A NEW SESSION:

1. Start a new Claude Code session (run `claude` in terminal)
2. Paste the HANDOFF CONTEXT above as your first message
3. Add your request: "Continue from the handoff context above. [Your next task]"

The new session will have all context needed to continue seamlessly.
```

---

## Important Constraints

- Do NOT attempt to programmatically create new sessions
- Provide a self-contained summary that works without access to the current session
- Use workspace-relative file paths
- Do NOT include sensitive information (API keys, credentials, secrets)
- FILES must not exceed 10 entries and must only reference accessible files
- Every section should contain information a new agent can act on or verify

---

## Execute Now

Begin by gathering context, then synthesize the handoff summary.
