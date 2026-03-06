---
name: tasknote-quick
description: "Quick task creation and updates using mdbase-tasknotes (mtn) CLI. Use for simple task operations: creating tasks, marking done, listing tasks, checking status. Triggers on 'create a task', 'add a task', 'mark done', 'what's on my plate', 'what's overdue', 'todo', 'remind me to'. For full lifecycle management with experiment logs, data logs, and meeting notes, use the tasknote skill instead."
---

# TaskNote Quick

Lightweight task operations using `mtn` CLI. For full lifecycle (experiment logs, data logs, meeting notes, multi-session tracking), use the `tasknote` skill instead.

## Before Anything: Resolve Collection Path

**CRITICAL: `mtn` must know which Obsidian vault to use.** Before running any `mtn` command:

1. **Read `CLAUDE.md`** (in the current project root or working directory) and look for a collection path (e.g., `MDBASE_TASKNOTES_PATH`, `collectionPath`, or a vault path).
2. If found, pass it to every `mtn` command with `-p <path>`:
   ```bash
   mtn list -p /path/to/vault
   mtn create "..." -p /path/to/vault
   ```
3. If `CLAUDE.md` does not specify a collection path, **ask the user** — do NOT let `mtn` fall back to its default config (which may point to a temp folder and waste time searching).

## Verify References Before Linking

**Never guess wikilink paths.** Before creating a task that references a project, verify it exists:

```bash
mtn search "<keyword>" -p <collection>
mtn list --json -p <collection>
```

**IMPORTANT: `mtn search` returns the note _title_, NOT the filename.** Titles can differ from filenames (e.g., title `AHA Scientific Session — Abstract Submission` vs filename `2026-03-05-PROJECT AHA Scientific Session Abstract Submission.md`). Obsidian wikilinks resolve by **filename**, not title. To get the actual filename:

```bash
# Use mtn show to get the Path:
mtn show "<title from search>" -p <collection>
# Look for the "Path:" line → extract filename without folder prefix and .md
```

Use the **filename** (without `.md`) in wikilinks: `[[2026-03-05-PROJECT AHA Scientific Session Abstract Submission]]`.

If the user says "for project BII" or "subtask of preprocessing", fuzzy-match against search results. If no match, show candidates and ask. Never invent a `[[path]]`.

## Create a Task

```bash
# 1. Create via mtn NLP — use +[[Note Name]] for project/parent linking
mtn create "<text> @context +[[Note Name With Spaces]] due <date> <priority> priority #tag"
# Output: → tasks/<Title>.md

# 2. Fix projects frontmatter — mtn double-wraps wikilinks, strip it
# Linux / Windows (Git Bash):
sed -i "s|\[\[projects/\[\[\(.*\)\]\]\]\]|[[\1]]|g" "tasks/<Title>.md"
# macOS:
sed -i '' "s|\[\[projects/\[\[\(.*\)\]\]\]\]|[[\1]]|g" "tasks/<Title>.md"

# 3. Rename to convention — use the EXACT output path from step 1
DATE=$(date +%Y-%m-%d)
mv "tasks/<Title>.md" "tasks/${DATE}-TASK <Title>.md"
```

**NLP patterns:** `#tag`, `@context` (data/experiment/code/writing/admin), `+[[Note Name]]` (project/parent link), date phrases, priority words, `~30m` estimate.

**Important:** `mtn` always wraps `+` values as `[[projects/...]]` and double-wraps `[[` brackets. The sed post-processing step fixes this. The `+[[Name]]` syntax works for both project and task parents — no need to include folder prefixes like `projects/` or `tasks/`.

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
