---
name: tasknote-quick
description: "Quick task creation and updates using mdbase-tasknotes (mtn) CLI. Use for simple task operations: creating tasks, marking done, listing tasks, checking status. Triggers on 'create a task', 'add a task', 'mark done', 'what's on my plate', 'what's overdue', 'todo', 'remind me to'. For full lifecycle management with experiment logs, data logs, and meeting notes, use the tasknote skill instead."
---

# TaskNote Quick

Lightweight task operations using `mtn` CLI. For full lifecycle (experiment logs, data logs, meeting notes, multi-session tracking), use the `tasknote` skill instead.

## Before Anything: Verify References

**CRITICAL: Never guess wikilink paths.** Before creating a task that references a project or parent task, verify it exists:

```bash
# Find matching projects/tasks
mtn search "<keyword>" -p <collection>
mtn list -p <collection>
```

If the user says "for project BII" or "subtask of preprocessing", fuzzy-match against search results. If no match, show candidates and ask. Never invent a `[[path]]`.

## Create a Task

```bash
# 1. Create via mtn NLP
mtn create "<text> @context +exact-project-name due <date> <priority> priority #tag" -p <collection>
# Output: → tasks/<Title>.md

# 2. Rename to convention
DATE=$(date +%Y-%m-%d)
mv "<collection>/tasks/<Title>.md" "<collection>/tasks/${DATE}-TASK <Title>.md"
```

**NLP patterns:** `#tag`, `@context` (data/experiment/code/writing/admin), `+project`, date phrases, priority words, `~30m` estimate.

After creation, edit the body to add a brief Log entry:

```markdown
## Log

### YYYY-MM-DD — Created
[What needs to be done and any context]
```

For substantial tasks, also add Motivation and Goals sections above the Log.

If the task has a parent, add to frontmatter after creation:
```yaml
parent: "[[tasks/YYYY-MM-DD-TASK Parent Task Name]]"
```

## Update a Task

```bash
mtn update "<task>" --status in-progress
mtn update "<task>" --priority high --due 2026-04-01
mtn update "<task>" --scheduled 2026-03-10    # calendar visibility
```

When adding progress, append a date-stamped Log entry to the body (don't overwrite existing entries).

## Complete a Task

```bash
mtn complete "<task>"
```

Optionally add a brief Summary section between Goals and Log before completing.

## List / Search

```bash
mtn list -p <collection>                    # Active tasks
mtn list --overdue -p <collection>          # Overdue
mtn search "<query>" -p <collection>        # Full-text search
mtn timer log --period week -p <collection> # Time this week
```

## Time Tracking

```bash
mtn timer start "<task>" -d "focus description" -p <collection>
mtn timer stop -p <collection>
```

## Filename Convention

All files: `YYYY-MM-DD-TYPE Title.md`. Types: `PROJECT`, `TASK`, `CARD`, `REF`, `DATA-LOG`, `EXP-LOG`, `MEETING`.

## Key Rules

- **Verify before linking** — search for projects/tasks before using their paths in wikilinks
- **One task per piece of work** — use Log entries for updates, not new task files
- **Rename after create** — `mtn create` then `mv` to `YYYY-MM-DD-TASK <title>.md`
- **Don't over-ask** — if intent is clear, just create the task
- **Read before edit** — always read a file before modifying it
