# mtn Command Reference

Quick reference for all `mtn` (mdbase-tasknotes) commands.

## Task Creation

```bash
mtn create "<natural language text>"
```

The NLP parser extracts:

| Pattern | Example | Extracted as |
|---|---|---|
| `#tag` | `#shopping` | Tag |
| `@context` | `@errands` | Context |
| `+project` | `+quarterly-review` | Project (stored as wikilink) |
| Date words | `tomorrow`, `friday`, `next week` | Due date |
| Priority words | `high priority`, `urgent` | Priority |
| Recurrence | `every day`, `weekly`, `every monday` | Recurrence rule |
| Estimate | `~30m`, `~2h` | Time estimate |

## Querying

```bash
# Default: open tasks ordered by due date
mtn list

# Filter by status, priority, tag, or context
mtn list --status in-progress
mtn list --priority high
mtn list --tag work

# Overdue tasks
mtn list --overdue

# Raw mdbase where expression
mtn list --where 'due < "2026-03-01" && priority == "urgent"'

# JSON output for scripting
mtn list --json
```

## Task Detail

```bash
mtn show "<task title or path>"
```

Title matching: exact match first, then substring.

## Search

```bash
mtn search <query>
```

Full-text search across titles, body content, tags, contexts, and projects.

## Updating

```bash
# Complete a task
mtn complete "<task>"

# Update fields
mtn update "<task>" --status in-progress
mtn update "<task>" --priority urgent --due 2026-03-01
mtn update "<task>" --add-tag important --remove-tag draft
mtn update "<task>" --add-context office

# Archive and delete
mtn archive "<task>"
mtn delete "<task>"
mtn delete "<task>" --force   # Skip backlink check
```

## Time Tracking

```bash
# Start a timer
mtn timer start "<task>"
mtn timer start "<task>" -d "Description of work"

# Check active timers
mtn timer status

# Stop the running timer
mtn timer stop

# View time log
mtn timer log
mtn timer log --period today
mtn timer log --period week
mtn timer log --from 2026-02-01 --to 2026-02-28
```

## Projects

```bash
# List all projects
mtn projects list
mtn projects list --stats    # With completion percentages

# Show tasks for a project
mtn projects show <project-name>
```

## Statistics

```bash
mtn stats
```

Shows total tasks, completion rate, overdue count, breakdown by status and priority, total time tracked.

## Configuration

```bash
mtn config --list
mtn config --set collectionPath=/path/to/vault
mtn config --get collectionPath
```

Collection path resolution order:
1. `--path` / `-p` flag on any command
2. `MDBASE_TASKNOTES_PATH` environment variable
3. `collectionPath` in config file
4. Current working directory

## File Operations

Since non-task notes (projects, logs, cards, meetings) aren't mtn tasks, create them directly:

```bash
# Create directories if needed
mkdir -p projects/ cards/detail/ cards/reference/ logs/ meetings/

# Create a file (use Write tool or bash)
# Then populate with the appropriate template from references/templates.md
```

For tasks, always use `mtn create`, then rename to the filename convention, then edit the body:

```bash
# 1. Create via mtn (generates title-based filename)
mtn create "My task title @context +project" -p <collection>
# Output: → tasks/My task title.md

# 2. Rename to convention: YYYY-MM-DD-TASK <title>.md
DATE=$(date +%Y-%m-%d)
mv "<collection>/tasks/My task title.md" \
   "<collection>/tasks/${DATE}-TASK My task title.md"

# 3. Edit the body (add Motivation, Goals, Log)
```

`mtn` indexes by frontmatter, not filename — renamed files are found normally by `mtn list`, `mtn search`, etc.

## Task File Structure

Each task is a markdown file with YAML frontmatter:

```markdown
---
tags:
  - task
  - project/2026-02-MICCAI
  - modality/cine
  - source/ukbb
title: Collect CMR cine data from UK Biobank
status: in-progress
priority: high
due: 2026-03-15
contexts:
  - data
projects:
  - "[[projects/2026-02-MICCAI]]"
parent: "[[tasks/Collect CMR data]]"
cards:
  - "[[cards/detail/CMR-cine-preprocessing]]"
  - "[[cards/reference/UK-Biobank-access]]"
timeEstimate: 240
timeEntries:
  - startTime: "2026-03-05T09:00:00.000Z"
    endTime: "2026-03-05T10:30:00.000Z"
    description: "Initial data download"
---

## Motivation
...

## Goals
- ...

## Log
### 2026-03-05 — Created
...
```

### Core Properties

| Property | Type | Example |
|---|---|---|
| title | Text | `title: Collect CMR data` |
| status | Enum | `status: in-progress` |
| priority | Enum | `priority: high` |
| due | Date | `due: 2026-03-15` |
| scheduled | DateTime | `scheduled: 2026-03-15T09:00` |
| contexts | List | `contexts: [data, code]` |
| projects | Link List | `projects: ["[[projects/2026-02-MICCAI]]"]` |
| tags | List | `tags: [modality/cine, source/ukbb]` |
| parent | Link | `parent: "[[tasks/Parent Task]]"` |
| blockedBy | Link List | `blockedBy: [{uid: "[[tasks/Other]]"}]` |
| cards | Link List | `cards: ["[[cards/detail/Something]]"]` |
| timeEstimate | Number | `timeEstimate: 60` (minutes) |

### Statuses (default)

open, in-progress, done, cancelled

### Priorities (default)

low, normal, high, urgent
