# TaskNote — Claude Code Plugin

A Claude Code plugin for managing research task lifecycle using [mdbase-tasknotes](https://github.com/jr-xing/mdbase-tasknotes) (`mtn`) CLI. Tracks tasks, experiments, data processing, and meetings in an Obsidian vault with markdown files.

## What It Does

This plugin provides **two skills**:

**`tasknote-quick`** — Lightweight task creation and updates. Use for everyday task operations: creating tasks, marking done, listing what's on your plate, checking overdue items. Fast and minimal.

**`tasknote`** — Full lifecycle management. Use when you need the complete research workflow: experiment logs with run tracking, data logs with pipeline tables, meeting notes with action items, multi-session tasks with time tracking, and project hierarchy management.

Both skills share the same conventions: proper project/task matching (always verifies wikilinks before creating them), filename convention (`YYYY-MM-DD-TYPE Title.md`), and prompt-first workflow.

All notes are plain markdown with YAML frontmatter, stored in your Obsidian vault. They work with both the `mtn` CLI and the [TaskNotes](https://github.com/jr-xing/tasknotes) Obsidian plugin.

## Prerequisites

- [mdbase-tasknotes](https://github.com/jr-xing/mdbase-tasknotes) (`mtn`) CLI installed
- An Obsidian vault (or any folder) initialized with `mtn init`

## Installation

### As a Claude Code plugin

```bash
claude plugin add /path/to/tasknote-skill
```

Or from GitHub:

```bash
claude plugin add github:jr-xing/tasknote-skill
```

### Per-repo setup

1. Create `.tasknote.yaml` in your repo root:

```yaml
collection: "/path/to/your/obsidian-vault"
projects:
  - "[[projects/2026-02-01-PROJECT Your Project Name]]"
```

2. Copy `CLAUDE.md.template` to your repo as `CLAUDE.md` (or append its contents to your existing `CLAUDE.md`). This tells Claude Code to use the task system during coding sessions.

## Skills

### tasknote-quick

For simple, everyday task operations (~80 lines). Triggers on: "create a task", "add a task", "mark done", "what's on my plate", "what's overdue", "todo", "remind me to".

Covers: create, update, complete, list/search, time tracking, filename convention.

### tasknote (full lifecycle)

For comprehensive research workflow management (~430 lines + references). Triggers on: "I need to...", "I just finished...", "the results are...", "update the data log", "create a meeting note", "what's the status of...".

Covers everything in `tasknote-quick`, plus: experiment logs, data logs, meeting notes, multi-session task tracking, detail/reference cards, project hierarchy, cross-updating between tasks and logs.

## How It Works

### Session Workflow

**On session start:** Claude reads `.tasknote.yaml`, lists active tasks for your projects, and matches your prompt to a task (or creates a new one). A timer starts automatically.

**During the session:** As you work, Claude appends log entries to the active task. Experiment results go to both the task and the experiment log. Data changes update the data log.

**On session end:** Claude stops the timer, writes a session summary, and asks when you plan to continue (updates `scheduled` for calendar visibility).

### Note Types

| Type | Prefix | Purpose |
|------|--------|---------|
| Project | `PROJECT` | Top-level research initiative (paper, grant) |
| Task | `TASK` | Specific piece of work with lifecycle |
| Detail Card | `CARD` | Technical details extracted from a task |
| Reference Card | `REF` | Permanent shared knowledge (access procedures, contacts) |
| Data Log | `DATA-LOG` | Per-dataset changelog with pipeline stage table |
| Experiment Log | `EXP-LOG` | Per-project run log with metrics tracking |
| Meeting | `MEETING` | Notes with decisions and linked action items |

All files follow the naming convention: `YYYY-MM-DD-TYPE Title.md`

### Prompt-First Workflow

You don't need to explicitly say "create a task." Just describe what you're doing:

- "I need to rerun training with the new data" → creates a task
- "The combined training finished, Dice LV=0.941" → updates existing task + experiment log
- "The preprocessing pipeline is done" → completes the task with a summary
- "Remind me to email Dr. Chen" → creates a quick task

### Reference Verification

Both skills verify project and task references before creating wikilinks. If you say "add a subtask for project BII", Claude will search existing projects, find the best match, and confirm with you rather than guessing a path that may not exist.

## Plugin Contents

```
tasknote-skill/
├── .claude-plugin/
│   ├── plugin.json                # Plugin metadata
│   └── marketplace.json           # Marketplace distribution config
├── skills/
│   ├── tasknote/                  # Full lifecycle skill
│   │   ├── SKILL.md
│   │   └── references/
│   │       ├── mtn-commands.md    # CLI command reference
│   │       ├── templates.md       # Note type templates
│   │       ├── taxonomy.md        # Tag hierarchy and context values
│   │       └── property-types.md  # Frontmatter property types/formats
│   └── tasknote-quick/            # Lightweight task skill
│       └── SKILL.md
├── CLAUDE.md.template             # Per-repo setup template
└── README.md
```

## License

MIT
