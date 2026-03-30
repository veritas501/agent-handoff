# agent-handoff

A Claude Code custom slash command for seamless cross-session context handoff.

## What is this?

When working with Claude Code, long sessions inevitably hit context window limits — responses degrade, context gets auto-compressed, and important details are lost. `/handoff` solves this by generating a structured context summary you can paste into a new session to continue work without losing anything.

It automatically collects git state, task progress, conversation history, and project configuration, then synthesizes everything into a portable plain-text handoff document.

## Features

- **Auto-collects git context** — branch, recent commits, diffs, uncommitted changes
- **Extracts session context** — user requests (verbatim), decisions, constraints, key files
- **Detects project config** — reads CLAUDE.md and TaskList if available
- **Structured output** — consistent format optimized for new session ingestion
- **Brief mode** — `--brief` flag for a condensed summary

## Install

### Via Plugin Marketplace (recommended)

```bash
# Add the marketplace
/plugin marketplace add veritas501/agent-handoff

# Install the plugin
/plugin install agent-handoff@agent-handoff-marketplace
```

### Manual - Project-level (single project)

```bash
mkdir -p .claude/commands
curl -o .claude/commands/handoff.md https://github.com/veritas501/agent-handoff/raw/refs/heads/master/plugins/agent-handoff/commands/handoff.md
```

## Usage

```
/agent-handoff:handoff              # Full context handoff summary
/agent-handoff:handoff --brief      # Condensed version (goal + state + tasks + files only)
```

If installed manually via `~/.claude/commands/`, the command is simply `/handoff`.

Then in a new Claude Code session, paste the output as your first message and add your next request.

## Output Format

The full handoff summary includes these sections:

| Section | Description |
|---------|-------------|
| USER REQUESTS (AS-IS) | Verbatim user requests from the session |
| GOAL | One sentence — what should be done next |
| WORK COMPLETED | What was accomplished (first person) |
| CURRENT STATE | Codebase state, branch, uncommitted changes |
| PENDING TASKS | Incomplete work and next steps |
| KEY FILES | Up to 10 most important files |
| IMPORTANT DECISIONS | Technical decisions and rationale |
| EXPLICIT CONSTRAINTS | Verbatim constraints from user or config |
| CONTEXT FOR CONTINUATION | Warnings, gotchas, references |

The `--brief` mode outputs only GOAL, CURRENT STATE, PENDING TASKS, and KEY FILES.

## Credits

Inspired by:

- [code-relay](https://github.com/yan5xu/code-relay) — structured cross-session HANDOFF/CHECKPOINT protocol for AI coding agents
- [oh-my-openagent](https://github.com/nicepkg/oh-my-openagent) — built-in handoff command template with phased execution flow
