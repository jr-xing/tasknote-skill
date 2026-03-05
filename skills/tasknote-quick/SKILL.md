---
name: tasknote-quick
description: "Quick task creation and updates using mdbase-tasknotes (mtn) CLI. Use for simple task operations: creating tasks, marking done, listing tasks, checking status. Triggers on 'create a task', 'add a task', 'mark done', 'what's on my plate', 'what's overdue', 'todo', 'remind me to'. For full lifecycle management with experiment logs, data logs, and meeting notes, use the tasknote skill instead."
---

# TaskNote Quick

Lightweight task operations using `mtn` CLI. For full lifecycle (experiment logs, data logs, meeting notes, multi-session tracking), use the `tasknote` skill instead.

## Before Anything: Verify References

**CRITICAL: Never guess wikilink paths.** Before creating a task that references a project, verify it exists:

```bash
# Find matching projects/tasks (no -p flag — mtn resolves collection automatically)
mtn search "<keyword>"
mtn list
```

If the user says "for project BII" or "subtask of preprocessing", fuzzy-match against search results. If no match, show candidates and ask. Never invent a `[[path]]`.

## Create a Task

```bash
# 1. Create via mtn NLP (no -p flag — mtn resolves collection automatically)
# Do NOT use +project — it breaks with spaces. Add projects via frontmatter edit instead.
mtn create "<text> @context due <date> <priority> priority #tag"
# Output: → tasks/<Title>.md

# 2. Rename to convention — use the EXACT output path from step 1
DATE=$(date +%Y-%m-%d)
mv "tasks/<Title>.md" "tasks/${DATE}-TASK <Title>.md"

# 3. Add project link — edit the task's frontmatter to add:
#    projects:
#      - "[[projects/YYYY-MM-DD-PROJECT Project Name]]"
```

**NLP patterns:** `#tag`, `@context` (data/experiment/code/writing/admin), date phrases, priority words, `~30m` estimate. **Note:** `+project` does not work reliably with spaces — always add projects via frontmatter edit.

After creation, edit the body to add a brief Log entry:

```markdown
## Log

### YYYY-MM-DD — Created
[What needs to be done and any context]
```

**Motivation and Goals sections are OPT-IN only.** Only add these if the user explicitly provides the content:
- Add `## Motivation` only if the user says something like "motivation:...", "motiv:...", "because...", or clearly states why the task matters
- Add `## Goals` only if the user explicitly lists goals (e.g., "goal:...", "goals are...", "deliverables:...")
- If the user's prompt does not contain motivation or goals, **do NOT add these sections and do NOT fabricate content for them** — just add the Log entry

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
mtn list                    # Active tasks
mtn list --overdue          # Overdue
mtn search "<query>"        # Full-text search
mtn timer log --period week # Time this week
```

## Time Tracking

```bash
mtn timer start "<task>" -d "focus description"
mtn timer stop
```

## Filename Convention

All files: `YYYY-MM-DD-TYPE Title.md`. Types: `PROJECT`, `TASK`, `CARD`, `REF`, `DATA-LOG`, `EXP-LOG`, `MEETING`.

## Key Rules

- **Verify before linking** — search for projects/tasks before using their paths in wikilinks
- **One task per piece of work** — use Log entries for updates, not new task files
- **Rename after create** — `mtn create` then `mv` to `YYYY-MM-DD-TASK <title>.md`
- **Don't over-ask** — if intent is clear, just create the task
- **Read before edit** — always read a file before modifying it
