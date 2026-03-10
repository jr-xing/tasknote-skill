---
name: tasknote
description: "Manage research task lifecycle using mdbase-tasknotes (mtn) CLI with hierarchical project/task/card organization. Use this skill whenever the user mentions tasks, todos, work items, experiments, data processing, or any research activity they want to track. Trigger on phrases like 'I need to...', 'I just finished...', 'what's overdue?', 'start working on...', 'the results are...', 'update the data log', 'create a meeting note', or 'what's the status of...'. Also trigger when the user describes new work to do (prompt-first workflow — the prompt itself can become a task). Covers the full lifecycle: project planning, task creation, progress tracking with time logging, experiment and data log management, meeting notes, and completion with summaries."
---

# TaskNote Lifecycle Skill

This skill manages a hierarchical research task system using `mtnj` (a fork of mdbase-tasknotes with improved project name handling and `--folder` support). Install: `npm install -g jr-xing/mdbase-tasknotes`. Each piece of work lives as a markdown file with YAML frontmatter — queryable metadata on top, rich human-readable content below.

The system has several note types organized in a hierarchy:

```
Project notes  →  Task notes  →  Cards (detail / reference)
     ↕               ↕
  Data logs     Experiment logs     Meeting notes
```

For the full mtn command reference, read `references/mtn-commands.md` in this skill's directory.
For note templates and frontmatter examples, read `references/templates.md`.
For the tag taxonomy and context values, read `references/taxonomy.md`.
For exact property names, types, and formats, read `references/property-types.md`.

## Prerequisites

`mtnj` must be installed (`npm install -g jr-xing/mdbase-tasknotes`). Before running any `mtnj` command, resolve the collection path:

1. **Read `CLAUDE.md`** (in the current project root or working directory) and look for a collection path (e.g., `MDBASE_TASKNOTES_PATH`, `collectionPath`, or a vault path).
2. If found, pass it to **every** `mtnj` command with `-p <path>`:
   ```bash
   mtnj list -p /path/to/vault
   mtnj create "..." -p /path/to/vault
   ```
3. If `CLAUDE.md` does not specify a collection path, **ask the user** — do NOT let `mtnj` fall back to its default config (which may point to a temp folder and waste time searching).

Never run `mtnj` without `-p` unless you have confirmed the collection path is correctly configured via `mtnj config --get collectionPath`.

---

## Prompt-First Workflow

The user's natural language IS the entry point. Your job is to figure out the intent and act:

**Clear task intent** → Create the task automatically.
"I need to rerun the segmentation training with Yale data for MICCAI" → Create task, set up body, start timer.

**Ambiguous intent** → Ask before creating.
"I wonder if we should try a different loss function" → "Would you like me to create a task to investigate alternative loss functions, or is this just a thought?"

**Progress report** → Find and update the existing task.
"The combined training finished, Dice LV=0.941" → Find the task, append log entry, update experiment log.

**Completion signal** → Summarize and close.
"The preprocessing pipeline is done and working" → Write summary, complete the task.

**Quick capture** → Lightweight task, minimal ceremony.
"Remind me to email Dr. Chen about T1 data" → Create task with just a title and log entry, no elaborate Motivation/Goals.

---

## Note Types

### 1. Project Note

Top-level anchor stored in `projects/`. Not a task — it describes scope, timeline, and milestones. Tasks link to projects via `projects` frontmatter; Obsidian backlinks automatically show all related tasks.

Create when: User starts a new research initiative (paper, grant, collaboration).

```bash
# Projects are plain markdown, not mtn tasks. Create the file directly.
```

Frontmatter:
```yaml
title: "MICCAI 2026 — Cardiac Segmentation with Foundation Models"
status: active                    # active | paused | completed | archived
tags:
  - project/2026-02-MICCAI
  - venue/miccai
deadline: 2026-06-15
```

Body: Objective, Key Milestones (checkboxes), Current Status section.

### 2. Task Note

The core work unit. Created via `mtnj create`, then body-enriched.

```bash
# Use +[[Note Name]] for project linking — works with spaces natively in mtnj
mtnj create "Collect CMR cine data from UK Biobank due march-15 high priority @data +[[2026-02-01-PROJECT MICCAI 2026]] #modality/cine #source/ukbb" --folder tasks -p <collection>
```

`mtnj` produces clean wikilinks in frontmatter — no post-processing needed. After creation, rename to convention. Use `mtnj show "<title>"` to find the file path, then edit the body.

> **Legacy note:** The original `mtn` CLI double-wraps wikilinks (`[[projects/[[...]]]]`) and lacks `--folder`. If using `mtn`, apply sed fix after creation: `sed -i "s|\[\[projects/\[\[\(.*\)\]\]\]\]|[[\1]]|g" "<file>"`

**Body structure for substantial tasks:**

```markdown
## Motivation
[1-3 sentences: why this matters]

## Goals
- [Concrete, verifiable deliverable 1]
- [Concrete, verifiable deliverable 2]

## Summary          ← added only on completion, placed here between Goals and Log

## Log
### YYYY-MM-DD — Created
[Initial context]
```

**For quick/lightweight tasks**, skip Motivation and Goals — just add a Log entry.

### 3. Detail Card

Extracts lengthy technical content from a task to keep it clean. Stored in `cards/detail/`.

Create when: A task's body would exceed ~20 lines of technical specification, or you need to document a reusable procedure.

```yaml
title: "CMR Cine Preprocessing Pipeline Details"
tags:
  - card/detail
parentTask: "[[tasks/Collect CMR cine data]]"
```

Link from the parent task's `cards` frontmatter field and reference inline: "See [[cards/detail/CMR-cine-preprocessing]] for full pipeline details."

### 4. Reference Card

Permanent, shared knowledge that multiple tasks or projects reference. Stored in `cards/reference/`.

Create when: Information is important enough to be its own document and is used by more than one task (data access procedures, environment setup, key contacts).

```yaml
title: "UK Biobank CMR Data Access"
tags:
  - card/reference
  - source/ukbb
relatedProjects:
  - "[[projects/2026-02-MICCAI]]"
lastVerified: 2026-03-01
```

### 5. Data Log

**Global** living document per dataset (or per modality-per-source). Not project-specific because datasets are often shared across projects. Stored in `logs/`.

Create when: User first works with a dataset, or mentions data status that should be permanently tracked.

```yaml
title: "Data Log — CMR Cine — UK Biobank"
tags:
  - log/data
  - modality/cine
  - source/ukbb
relatedProjects:
  - "[[projects/2026-02-MICCAI]]"
  - "[[projects/2026-03-AHA-Fellowship]]"
```

Body structure:
```markdown
## Current State
| Stage | Status | Count | Location | Last Updated |
|-------|--------|-------|----------|-------------|
| Raw DICOM | Complete | 45,000 | /data/ukbb/raw/ | 2026-02-20 |
| NIfTI conversion | Complete | 44,850 | /data/ukbb/nifti/ | 2026-02-25 |
| Segmentation | Pending | — | — | — |

## Changelog
### YYYY-MM-DD — [What changed]
[Details of what was done, counts, any issues]
```

Always keep the "Current State" table up to date when adding changelog entries.

### 6. Experiment Log

**Per-project** living document for a series of related experiment runs. Stored in `logs/`.

Create when: User begins a series of experiments for a specific research question within a project.

```yaml
title: "Experiment Log — Cardiac Segmentation — MICCAI 2026"
tags:
  - log/experiment
  - model/segmentation
  - project/2026-02-MICCAI
relatedProjects:
  - "[[projects/2026-02-MICCAI]]"
```

Body structure:
```markdown
## Best Result So Far
| Metric | Value | Run | Date |
|--------|-------|-----|------|
| Dice (LV) | 0.943 | run-047 | 2026-02-28 |

## Runs
### run-052 — YYYY-MM-DD
**Config:** [model, batch, lr, epochs, data]
**Changes:** [What's different from previous run]
**Result:** [Key metrics]
**Notes:** [Observations, issues, ideas]
**Wandb/Link:** [optional tracking link]
```

Keep "Best Result So Far" updated whenever a run beats the previous best.

### 7. Meeting Note

Cross-cutting notes from PI meetings, collaborations. Stored in `meetings/`.

```yaml
title: "PI Meeting — 2026-03-05"
tags:
  - meeting
  - meeting/pi
contexts:
  - meeting
date: 2026-03-05
relatedProjects:
  - "[[projects/2026-02-MICCAI]]"
actionItems:
  - "[[tasks/Rerun segmentation with Yale data]]"
```

Body: Discussion, Decisions, Action Items (with links to created tasks).

After writing a meeting note, create tasks for each action item and link them in the `actionItems` field.

---

## Task Lifecycle

### Phase 1: Creation

1. **Resolve references FIRST** — before creating anything, verify that projects and blockers actually exist:
   ```bash
   # Search for a specific project/task
   mtnj search "<keyword>" -p <collection>
   # Or list with JSON to get file paths
   mtnj list --json -p <collection>
   ```
   - **`mtnj search` returns the note _title_, NOT the filename.** Titles can differ from filenames (e.g., title `AHA Scientific Session — Abstract Submission` vs filename `2026-03-05-PROJECT AHA Scientific Session Abstract Submission.md`). To get the actual filename, run `mtnj show "<title>" -p <collection>` and look for the `Path:` line.
   - Obsidian wikilinks resolve by **filename**, not title. Use the filename (without folder prefix and `.md`): `[[2026-03-05-PROJECT AHA Scientific Session Abstract Submission]]`.
   - If the user says "for project BII" or "subtask of preprocessing", fuzzy-match against existing notes. **Never guess a wikilink path** — always verify first.
   - If no match is found, ask the user: "I couldn't find a project matching 'BII'. Did you mean '2026-03-05-PROJECT Yale Biomedical Imaging Institute Project List'?" Show candidates.
   - For `projects`, the target must be an existing project note.
2. **Create via `mtnj create`** with NLP patterns and `--folder` to control output directory:
   - `#tag` for tags (use hierarchical: `#modality/cine`, `#source/ukbb`)
   - `@context` for context (`@data`, `@experiment`, `@code`, `@writing`)
   - `+[[Note Name]]` for project/parent link (works with spaces natively)
   - Date phrases, priority words, `~estimate`
   - `--folder tasks` (or `projects`, etc.) to place the file in the right directory
   - `mtnj` produces clean wikilinks — no sed post-processing needed.
3. **Rename** the file to the filename convention (see Filename Convention section).
4. **Enrich the body** — read the file with `mtnj show`, then add a Log entry. Only add Motivation/Goals sections if the user explicitly provided that content — never fabricate them.
5. **Start timer** if the user is beginning work now.
6. **Cross-reference** — if this task relates to an existing data or experiment log, mention the connection in the log entry.

**Context inference guide:**
- Mentions of downloading, cleaning, preprocessing, annotation → `@data`
- Mentions of training, tuning, evaluation, metrics → `@experiment`
- Mentions of coding, debugging, refactoring, pipeline → `@code`
- Mentions of drafting, writing, editing, manuscript → `@writing`
- Mentions of planning, design, architecture, scope → `@plan`
- Mentions of email, access request, IRB, paperwork → `@admin`

**Tag inference guide:**
- Mentions of cine, LGE, T1, T2 → `#modality/cine`, `#modality/lge`, etc.
- Mentions of UK Biobank, UKBB → `#source/ukbb`
- Mentions of Yale, UPenn → `#source/yale`, `#source/upenn`
- Mentions of segmentation, classification → `#model/segmentation`, `#model/classification`

### Phase 2: Progress Updates

1. **Find the task** — use `mtnj search` or `mtnj list` to locate it.
2. **Update status** to `in-progress` if not already.
3. **Start/stop timer** based on whether the user is actively working vs. just reporting.
4. **Append to Log** — date-stamped entry with substantive content (decisions, results, blockers).
5. **Cross-update logs** when appropriate:
   - If user reports experiment results → also update the experiment log with a new run entry.
   - If user reports data processing completion → also update the data log's Current State table and add a changelog entry.
6. **Create follow-up tasks** when the user implies next steps ("I need to also..." or "next we should...").

### Phase 3: Completion

1. **Stop timer**.
2. **Write Summary section** — placed between Goals and Log. Synthesize the log entries into a coherent narrative with Key Outcomes and Lessons/Notes subsections.
3. **Complete via `mtnj complete`**.
4. **Update related logs** — if this was an experiment task, make sure the experiment log has the final results. If data task, make sure the data log's Current State reflects the new status.
5. **Check for unblocked tasks** — if other tasks had `blockedBy` pointing to this one, notify the user.

---

## Multi-Session Tasks

Many research tasks span days or weeks — training runs, data collection, paper writing. These tasks naturally require multiple work sessions. The system handles this without splitting into subtasks:

**Each work session produces two things in the same task note:**
1. A `timeEntry` in frontmatter (via `mtnj timer start/stop`) — the quantitative record
2. A Log entry in the body — the narrative record

The timeEntries array accumulates automatically:
```yaml
timeEntries:
  - startTime: "2026-03-05T09:00:00.000Z"
    endTime: "2026-03-05T11:30:00.000Z"
    description: "Architecture exploration"
  - startTime: "2026-03-07T14:00:00.000Z"
    endTime: "2026-03-07T16:45:00.000Z"
    description: "Implemented data loader"
  - startTime: "2026-03-10T10:00:00.000Z"
    endTime: "2026-03-10T12:00:00.000Z"
    description: "Memory fixes and testing"
```

**For calendar visibility**, use the `scheduled` field as a rolling "next session" marker. After each session, update it to when the user plans to work on this next:

```bash
mtnj timer stop
mtnj update "Train segmentation model" --scheduled 2026-03-12
```

**For at-a-glance progress**, add a phase marker at the top of the Log section:

```markdown
## Log

**Phase: Implementation** (Exploration → Design → **Implementation** → Evaluation → Write-up)
```

Update the phase marker as the task progresses through its natural stages.

### Session Workflow

**Starting a session:**
```bash
mtnj update "<task>" --status in-progress    # if not already
mtnj timer start "<task>" -d "Brief focus description"
```
Then append a new log entry header: `### YYYY-MM-DD — [Session label]`

**Ending a session:**
```bash
mtnj timer stop
mtnj update "<task>" --scheduled YYYY-MM-DD   # next planned session
```
Then fill in the log entry with what was accomplished, decisions, blockers, next steps.

**Resuming after days away:**
When the user comes back to a task after a gap, briefly review the last log entry before starting. If relevant, mention what changed since the last session (e.g., "picking up from where we left off on March 7 — the data loader is done, now working on the training loop").

**When to use detail cards instead of long log entries:**
If a single session produces very detailed technical output (>20 lines of specs, long code explanations, extensive experiment configs), extract it to a detail card and reference it from the log entry: "See [[cards/detail/training-config-v3]] for full hyperparameter sweep setup."

---

## Querying

| User says | What to do |
|---|---|
| "What's on my plate?" | `mtnj list` |
| "What's overdue?" | `mtnj list --overdue` |
| "What data tasks are there?" | `mtnj list --where 'contexts contains "data"'` |
| "Show experiments for MICCAI" | `mtnj list --where 'contexts contains "experiment"'` then filter by project tag |
| "How's the MICCAI paper going?" | `mtnj projects show 2026-02-MICCAI` + read the project note |
| "What's the current state of the cine data?" | Read the relevant data log file |
| "Find that thing about segmentation" | `mtnj search segmentation` |
| "How much time this week?" | `mtnj timer log --period week` |
| "What came out of the PI meeting?" | Search for recent meeting notes |

---

## Filename Convention

All note files follow a consistent naming pattern: `<date>-<TYPE> <title>.md`

| Note Type | Prefix | Example Filename |
|-----------|--------|-----------------|
| Project | `PROJECT` | `2026-02-01-PROJECT MICCAI 2026.md` |
| Task | `TASK` | `2026-03-05-TASK Train baseline nnU-Net.md` |
| Detail card | `CARD` | `2026-03-05-CARD nnunet-training-config.md` |
| Reference card | `REF` | `2026-02-15-REF UK-Biobank-CMR-access.md` |
| Data log | `DATA-LOG` | `2026-02-20-DATA-LOG CMR Cine UK Biobank.md` |
| Experiment log | `EXP-LOG` | `2026-02-28-EXP-LOG Cardiac Segmentation MICCAI 2026.md` |
| Meeting | `MEETING` | `2026-03-04-MEETING PI Meeting.md` |

The date is the creation date (`YYYY-MM-DD`). The title in frontmatter is unchanged — this only affects the filename.

**For tasks created via `mtnj create`**, the CLI generates a title-based filename. Rename immediately after creation:

```bash
# 1. Create the task (use +[[Name]] and --folder to control output)
mtnj create "Train baseline segmentation model due april-15 @experiment +[[2026-02-01-PROJECT MICCAI 2026]]" --folder tasks -p <collection>
# mtnj outputs: → tasks/Train baseline segmentation model.md
# No sed fix needed — mtnj produces clean wikilinks

# 2. Rename to convention — use the EXACT output path from step 1
DATE=$(date +%Y-%m-%d)
mv "tasks/Train baseline segmentation model.md" \
   "tasks/${DATE}-TASK Train baseline segmentation model.md"
```

**For non-task notes** (projects, logs, cards, meetings), create the file directly with the correct filename — no rename needed since these aren't created via `mtnj create`.

**Wikilinks use the filename** (without folder prefix or `.md` — Obsidian resolves the path). Note: filenames may differ from titles (e.g., em dashes `—` in titles may be absent in filenames). Always use `mtnj show` to get the actual filename:
```yaml
projects:
  - "[[2026-02-01-PROJECT MICCAI 2026]]"
cards:
  - "[[2026-03-05-CARD nnunet-training-config]]"
```

---

## Important Behaviors

**ALWAYS verify before linking.** Never guess wikilink paths for projects or blockers. Run `mtnj search` to find a match, then `mtnj show "<title>"` to get the actual **filename** (title ≠ filename). Use the filename (without folder prefix and `.md`) in wikilinks. If you can't find a match, ask the user — don't invent a path.

**Be proactive about status transitions.** "I'm starting on X" → set in-progress. "X is done" → complete. Don't wait to be explicitly told.

**Timestamps matter.** Always use actual current date in log entries (`date +%Y-%m-%d`).

**Don't over-ask for clear intents.** If the user says "I need to retrain with Yale data for MICCAI", just create the task. Don't ask "which project?" when they already said MICCAI — but DO verify the project path exists via search before linking.

**Do ask for ambiguous intents.** "Maybe we should try X" or "I was thinking about Y" — ask whether to track as a task.

**Preserve existing content.** When appending to Log sections, never overwrite or reorder previous entries.

**Read before writing.** Always read the file before editing it.

**Use mtnj's NLP.** Lean on the parser for dates, priorities, tags, contexts, and projects.

**Cross-update logs.** When a task update contains data status changes or experiment results, don't just update the task — update the relevant log too. This is important because logs are the permanent record while tasks are transient.

**Keep logs current.** The "Current State" table in data logs and the "Best Result" table in experiment logs should always reflect reality. When adding entries, also update the summary tables.
