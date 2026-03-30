---
description: Generate a structured cross-session context handoff summary for seamless continuation in a new session
argument-hint: [--brief]
---

# Handoff Command

## Purpose

Use /handoff when the session context is getting too long, quality is degrading, or you want to start fresh while preserving essential context. This generates a structured context summary that can be pasted into a new session for seamless continuation.

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

Write the context summary from the user's first-person perspective ("I did...", "I told you...", "I found...").

Extraction focus:
- Emphasize capabilities and behavior, not file-by-file implementation details
- Only retain information important for continuing the work
- Avoid excessive implementation details (variable names, storage keys, constants) unless critical
- USER REQUESTS must be quoted verbatim, never paraphrased
- EXPLICIT CONSTRAINTS must be quoted verbatim, never fabricated

Guiding questions for extraction:
- What was just done or implemented?
- Which instructions are still relevant (e.g., follow patterns in the codebase)?
- Which files were marked as important or are being modified?
- Is there a plan or spec that should be included?
- What important information was already communicated (libraries, patterns, constraints, preferences)?
- What important technical details were discovered (APIs, methods, patterns)?
- What caveats, limitations, or open questions were found?

### PHASE 4: Format Output

Check if `$ARGUMENTS` contains `--brief`.

**If `--brief` mode**, output the condensed version only:

```
HANDOFF CONTEXT (BRIEF)
=======================

GOAL
----
[One sentence describing what should be done next]

CURRENT STATE
-------------
- [Current state of the codebase or task]
- [Build/test status if applicable]
- Branch: [branch name] @ [commit SHA]

PENDING TASKS
-------------
- [Tasks that were planned but not completed]
- [Next logical steps to take]
- [Any blockers or issues encountered]

KEY FILES
---------
- [path/to/file1] - [brief role description]
- [path/to/file2] - [brief role description]
(Maximum 10 files, prioritized by importance)
```

**If full mode (default)**, output the complete version:

```
HANDOFF CONTEXT
===============

USER REQUESTS (AS-IS)
---------------------
- [Exact verbatim user requests - NOT paraphrased]

GOAL
----
[One sentence describing what should be done next]

WORK COMPLETED
--------------
- [First person bullet points of what was done]
- [Include specific file paths when relevant]
- [Note key implementation decisions]

CURRENT STATE
-------------
- [Current state of the codebase or task]
- [Build/test status if applicable]
- [Any environment or configuration state]
- Branch: [branch name] @ [commit SHA]
- Uncommitted changes: [yes/no, brief description]

PENDING TASKS
-------------
- [Tasks that were planned but not completed]
- [Next logical steps to take]
- [Any blockers or issues encountered]
- [Current task status from TaskList if available]

KEY FILES
---------
- [path/to/file1] - [brief role description]
- [path/to/file2] - [brief role description]
(Maximum 10 files, prioritized by importance; include files from git diff/status)

IMPORTANT DECISIONS
-------------------
- [Technical decisions that were made and why]
- [Trade-offs that were considered]
- [Patterns or conventions established]

EXPLICIT CONSTRAINTS
--------------------
- [Verbatim constraints only - from user or project configuration]
- If none, write: None

CONTEXT FOR CONTINUATION
------------------------
- [What the next session needs to know to continue]
- [Warnings or gotchas to be aware of]
- [References to documentation if relevant]
```

### Output Rules

- Plain text with bullets format
- Use `===` and `---` underline style for headers, no `#` markdown headers
- No bold, italic, or code fences within content
- Use workspace-relative paths for files
- Only include information that matters for continuation
- Pick an appropriate length based on complexity
- USER REQUESTS (AS-IS) and EXPLICIT CONSTRAINTS must contain verbatim quotes only

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
- KEY FILES must not exceed 10 entries
- GOAL should be a single sentence or short paragraph

---

## Execute Now

Begin by gathering context, then synthesize the handoff summary.
