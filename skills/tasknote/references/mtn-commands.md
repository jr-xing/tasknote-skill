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
| `+project` or `+[[Name]]` | `+quarterly-review` or `+[[2026-03-05-PROJECT Name]]` | Project link — use `+[[Name]]` for names with spaces (`mtnj` handles this natively) |
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

## Tree View

```bash
# Hierarchical project → task → subtask display
mtnj tree

# With filters
mtnj tree --status in-progress
mtnj tree --priority high
mtnj tree --tag source/yale
mtnj tree --overdue

# Include completed tasks (hidden by default)
mtnj tree --all
```

Shows tasks organized by project, with subtask nesting via box-drawing characters. Subtask relationships are determined by the `projects` field: if a task's `projects` wikilink points to another task (not a project), it becomes a subtask. Tasks with no project appear under "Orphan tasks". Project names are omitted from task lines since the hierarchy already shows them.

## Organize

```bash
# Preview planned moves (dry-run, default)
mtnj organize

# Execute the moves
mtnj organize --apply

# Also move orphan tasks to _unassigned/
mtnj organize --apply --orphans unassigned
```

Reorganizes task files into project folders based on their `projects` wikilink hierarchy. Tasks with children get their own subfolder; leaf tasks stay as plain files. Nesting depth follows the parent chain with no limit. Uses `collection.rename()` to update all wikilinks automatically. Idempotent — running again after organize shows "0 moves planned".

## Projects

```bash
# List all projects
mtn projects list

# Show tasks for a project (use the project title, not filename)
mtn projects show "<project title>"
```

**Note:** `mtn projects show` accepts the project **title** (from `mtn search` or `mtn projects list`), not the filename. This is the fastest way to list all subtasks of a project.

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

**Important:** Always use `-p <path>` explicitly. The default config may point to a temp folder, causing `mtn` to waste time searching. Read `CLAUDE.md` for the correct vault path; if not found, ask the user.

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
# 1. Create via mtn (use +[[Name]] for project/parent linking)
mtn create "My task title @context +[[2026-03-05-PROJECT My Project]]"
# Output: → tasks/My task title.md

# 2. Fix projects frontmatter — mtn double-wraps wikilinks
# Linux / Windows (Git Bash):
sed -i "s|\[\[projects/\[\[\(.*\)\]\]\]\]|[[\1]]|g" "tasks/My task title.md"
# macOS:
sed -i '' "s|\[\[projects/\[\[\(.*\)\]\]\]\]|[[\1]]|g" "tasks/My task title.md"

# 3. Rename to convention — use the EXACT output path from step 1
DATE=$(date +%Y-%m-%d)
mv "tasks/My task title.md" \
   "tasks/${DATE}-TASK My task title.md"

# 4. Edit the body (add Log entry; add Motivation/Goals only if user provided them)
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
  - "[[2026-02-MICCAI]]"
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
| projects | Link List | `projects: ["[[2026-02-MICCAI]]"]` |
| tags | List | `tags: [modality/cine, source/ukbb]` |
| blockedBy | Link List | `blockedBy: [{uid: "[[tasks/Other]]"}]` |
| cards | Link List | `cards: ["[[cards/detail/Something]]"]` |
| timeEstimate | Number | `timeEstimate: 60` (minutes) |

### Statuses (default)

open, in-progress, done, cancelled

### Priorities (default)

low, normal, high, urgent
