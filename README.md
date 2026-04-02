# agent-handoff

A Claude Code skill for seamless cross-session context handoff.

## What is this?

When working with Claude Code, long sessions inevitably hit context window limits — responses degrade, context gets auto-compressed, and important details are lost. `/handoff` solves this by generating a structured context summary you can paste into a new session to continue work without losing anything.

It automatically collects git state, task progress, conversation history, and project configuration, then synthesizes everything into a portable plain-text handoff document.

## Why Handoff?

Claude Code has a built-in mechanism called **compact** — when the context window fills up, the system automatically compresses earlier conversation turns. This is lossy and invisible: you don't control what gets kept or discarded, and the agent silently loses details it needed.

Handoff takes a different approach:

| | Compact | Handoff |
|---|---|---|
| **Trigger** | Automatic, when context is full | Manual, when you decide |
| **Output** | Hidden internal compression | Visible, editable plain text |
| **Control** | None — system decides what to keep | Full — you review before pasting |
| **Audience** | Same agent, degraded context | New agent, fresh context window |
| **Losses** | Unpredictable | None — structured extraction |
| **Lessons learned** | Lost with compressed turns | Preserved in GOTCHAS ("Tried X, failed because Y, fix was Z") |

The key insight: compact tries to keep a dying session alive. Handoff **starts a clean session with curated context** — the new agent gets exactly what it needs, nothing it doesn't, and learns from the previous agent's mistakes.

## Features

- **Auto-collects git context** — branch, recent commits, diffs, uncommitted changes
- **Extracts session context** — user requests (verbatim), decisions, constraints, key files
- **Detects project config** — reads CLAUDE.md and TaskList if available
- **Structured output** — consistent format optimized for new session ingestion
- **Brief mode** — `--brief` flag for a condensed summary

## Install

### Recommended: `npx skills`

```bash
npx skills add veritas501/agent-handoff
```

After installation, tell Claude Code:

```text
/handoff
```

Or simply say:

```text
generate a handoff summary
```

### Alternative: clone directly into Claude Code skills

```bash
git clone https://github.com/veritas501/agent-handoff.git ~/.claude/skills/agent-handoff
```

Claude Code discovers it automatically.

### Alternative: symlink for development

```bash
git clone https://github.com/veritas501/agent-handoff.git ~/code/agent-handoff
mkdir -p ~/.claude/skills
ln -s ~/code/agent-handoff ~/.claude/skills/agent-handoff
```

### Manual — single project

Download `SKILL.md` into your project's `.claude/commands/` directory:

```bash
mkdir -p .claude/commands
curl -o .claude/commands/handoff.md https://github.com/veritas501/agent-handoff/raw/refs/heads/master/SKILL.md
```

Then use `/handoff` in that project.

## Usage

```
/handoff              # Full context handoff summary
/handoff --brief      # Condensed version (goal + state + tasks + files only)
```

Then in a new Claude Code session, paste the output as your first message and add your next request.

## Output Format

The full handoff summary includes these sections:

| Section | Description |
|---------|-------------|
| USER REQUESTS (AS-IS) | Verbatim user requests from the session |
| TASK | Concrete next action for the new agent |
| COMPLETED | Verifiable facts about what was done |
| LAST OPERATIONS | Last 3-5 concrete operations as few-shot examples for the new agent |
| STATE | Branch, commit, uncommitted changes, build status |
| PENDING | Concrete tasks with enough detail to execute |
| FILES | Up to 10 accessible files with action context |
| RULES | Direct imperatives derived from decisions and constraints |
| GOTCHAS | Specific pitfalls, mistakes made, and non-obvious behaviors |

The `--brief` mode outputs only TASK, STATE, PENDING, and FILES.

## Credits

Inspired by:

- [code-relay](https://github.com/yan5xu/code-relay) — structured cross-session HANDOFF/CHECKPOINT protocol for AI coding agents
- [oh-my-openagent](https://github.com/nicepkg/oh-my-openagent) — built-in handoff command template with phased execution flow
