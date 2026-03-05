# TaskNote — Claude Code Plugin

A Claude Code plugin for managing research task lifecycle using [mdbase-tasknotes](https://github.com/jxing/mdbase-tasknotes) (`mtn`) CLI. Tracks tasks, experiments, data processing, and meetings in an Obsidian vault with markdown files.

## What It Does

When installed, Claude Code gains the ability to:

- **Create and manage tasks** from natural language — "I need to retrain the segmentation model with Yale data" becomes a properly tagged task with time tracking
- **Track multi-session work** across days/weeks with per-session time entries and log narratives
- **Maintain experiment logs** with run configs, metrics, and best-result tracking
- **Maintain data logs** with pipeline stage tables and changelogs
- **Write meeting notes** with decisions, action items linked to tasks
- **Follow a project hierarchy** — Project → Task → Cards (detail/reference), with cross-linked logs

All notes are plain markdown with YAML frontmatter, stored in your Obsidian vault. They work with both the `mtn` CLI and the [TaskNotes](https://github.com/jxing/tasknotes) Obsidian plugin.

## Prerequisites

- [mdbase-tasknotes](https://github.com/jxing/mdbase-tasknotes) (`mtn`) CLI installed
- An Obsidian vault (or any folder) initialized with `mtn init`

## Installation

### As a Claude Code plugin

```bash
claude plugin add /path/to/tasknote-skill
```

Or from GitHub:

```bash
claude plugin add github:jxing/tasknote-skill
```

### Per-repo setup

1. Create `.tasknote.yaml` in your repo root:

```yaml
collection: "/path/to/your/obsidian-vault"
projects:
  - "[[projects/2026-02-01-PROJECT Your Project Name]]"
```

2. Copy `CLAUDE.md.template` to your repo as `CLAUDE.md` (or append its contents to your existing `CLAUDE.md`). This tells Claude Code to use the task system during coding sessions.

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

## Skill Contents

```
skills/tasknote/
├── SKILL.md                      # Main skill instructions (~420 lines)
└── references/
    ├── mtn-commands.md           # CLI command reference
    ├── templates.md              # Copy-paste templates for all note types
    ├── taxonomy.md               # Tag hierarchy and context values
    └── property-types.md         # Frontmatter property types and formats
```

## License

MIT
