# Note Templates

Copy-paste ready templates for each note type. When creating notes, follow these structures.

## Table of Contents
1. [Project Note](#project-note)
2. [Task Note (Substantial)](#task-note-substantial)
3. [Task Note (Quick)](#task-note-quick)
4. [Task Note (Multi-Session)](#task-note-multi-session)
5. [Detail Card](#detail-card)
6. [Reference Card](#reference-card)
7. [Data Log](#data-log)
8. [Experiment Log](#experiment-log)
9. [Meeting Note](#meeting-note)

---

## Project Note

File location: `projects/YYYY-MM-DD-PROJECT [project-name].md`

```markdown
---
title: "[Project Title — Venue/Grant Year]"
status: active
tags:
  - project/[year-month-shortname]
  - venue/[venue-name]
deadline: YYYY-MM-DD
---

## Objective

[1-2 paragraph research goal. What are we trying to achieve and why?]

## Key Milestones

- [ ] [Milestone 1] — [Target date]
- [ ] [Milestone 2] — [Target date]
- [ ] [Milestone 3] — [Target date]
- [ ] [Final deliverable] — [Deadline]

## Current Status

[Updated periodically. 2-3 sentences on where things stand right now.]
```

---

## Task Note (Substantial)

Created via `mtn create`, then renamed and body added. Filename: `tasks/YYYY-MM-DD-TASK [title].md`

```markdown
---
title: "[Task title]"
status: open
priority: [low|normal|high|urgent]
due: YYYY-MM-DD
contexts:
  - [data|experiment|code|writing|plan|admin|review|meeting]
projects:
  - "[[projects/project-name]]"
tags:
  - project/[project-shortname]
  - [domain-specific tags]
parent: "[[tasks/Parent Task Name]]"     # optional, for subtasks
cards:                                    # optional, links to detail/reference cards
  - "[[cards/detail/card-name]]"
  - "[[cards/reference/card-name]]"
---

## Motivation

[1-3 sentences: why this task matters, what prompted it]

## Goals

- [Concrete, verifiable deliverable or outcome 1]
- [Concrete, verifiable deliverable or outcome 2]

## Log

### YYYY-MM-DD — Created
[Brief context: what prompted this task, initial decisions or constraints]
```

After completion, insert Summary between Goals and Log:

```markdown
## Summary

[2-4 sentence synthesis of what was accomplished]

### Key Outcomes
- [Outcome 1]
- [Outcome 2]

### Lessons / Notes
- [Anything worth remembering for future similar tasks]
```

---

## Task Note (Quick)

For lightweight tasks where Motivation/Goals would be overkill. Filename: `tasks/YYYY-MM-DD-TASK [title].md`

```markdown
---
title: "[Task title]"
status: open
priority: normal
contexts:
  - [context]
projects:
  - "[[projects/project-name]]"
tags:
  - project/[project-shortname]
---

## Log

### YYYY-MM-DD — Created
[What needs to be done and any relevant context]
```

---

## Task Note (Multi-Session)

For tasks that span multiple work sessions over days or weeks. Same filename as substantial task: `tasks/YYYY-MM-DD-TASK [title].md`. Uses a phase marker and rolling scheduled date.

```markdown
---
title: "[Task title]"
status: in-progress
priority: [priority]
due: YYYY-MM-DD
scheduled: YYYY-MM-DD              # Rolling: always set to NEXT planned session
contexts:
  - [context]
projects:
  - "[[projects/project-name]]"
tags:
  - project/[project-shortname]
  - [domain tags]
timeEntries:                        # Accumulates automatically via mtn timer
  - startTime: "2026-03-05T09:00:00.000Z"
    endTime: "2026-03-05T11:30:00.000Z"
    description: "Session 1 focus"
  - startTime: "2026-03-07T14:00:00.000Z"
    endTime: "2026-03-07T16:45:00.000Z"
    description: "Session 2 focus"
---

## Motivation
[1-3 sentences]

## Goals
- [Deliverable 1]
- [Deliverable 2]

## Log

**Phase: [Current Phase]** (Phase1 → Phase2 → **Phase3** → Phase4)

### YYYY-MM-DD — Session N: [Label]
[What was done, decisions, blockers]
Next session plan: [what to do next]

### YYYY-MM-DD — Session 1: [Label]
[Initial work]

### YYYY-MM-DD — Created
[Initial context]
```

The key patterns:
- `scheduled` always points to the next planned work session → visible on calendar
- `timeEntries` accumulate per session via `mtn timer start/stop` → time tracking
- Phase marker at top of Log → at-a-glance progress
- Each Log entry = one work session → coherent narrative
- If a session produces >20 lines of technical detail, extract to a detail card

---

## Detail Card

File location: `cards/detail/YYYY-MM-DD-CARD [descriptive-name].md`

```markdown
---
title: "[Descriptive Title]"
tags:
  - card/detail
parentTask: "[[tasks/Parent Task Name]]"
---

[Detailed technical content, specifications, step-by-step procedures, etc.
This content was extracted from the parent task to keep it clean.]
```

---

## Reference Card

File location: `cards/reference/YYYY-MM-DD-REF [descriptive-name].md`

```markdown
---
title: "[Descriptive Title]"
tags:
  - card/reference
  - [domain tags]
relatedProjects:
  - "[[projects/project-1]]"
  - "[[projects/project-2]]"
lastVerified: YYYY-MM-DD
---

[Permanent, shared knowledge. Data access procedures, environment setup,
key contacts, institutional protocols, etc.]
```

---

## Data Log

File location: `logs/YYYY-MM-DD-DATA-LOG [Modality] [Source].md`

Data logs are **global** (not per-project) because datasets are shared across projects.

```markdown
---
title: "Data Log — [Modality] — [Source]"
tags:
  - log/data
  - modality/[modality]
  - source/[source]
relatedProjects:
  - "[[projects/project-1]]"
  - "[[projects/project-2]]"
---

## Current State

| Stage | Status | Count | Location | Last Updated |
|-------|--------|-------|----------|-------------|
| Raw acquisition | [Status] | [N] | [path] | YYYY-MM-DD |
| Format conversion | [Status] | [N] | [path] | YYYY-MM-DD |
| Quality control | [Status] | [N] | [path] | YYYY-MM-DD |
| Preprocessing | [Status] | [N] | [path] | YYYY-MM-DD |
| Segmentation | [Status] | [N] | [path] | YYYY-MM-DD |
| Annotation | [Status] | [N] | [path] | YYYY-MM-DD |

Status values: Complete, In Progress, Pending, Blocked, N/A

## Changelog

### YYYY-MM-DD — [Brief description of change]
[What was done, counts affected, any issues or notes.
Reference the task that triggered this if applicable.]
```

---

## Experiment Log

File location: `logs/YYYY-MM-DD-EXP-LOG [Series Name] [Project].md`

Experiment logs are **per-project** because experiments are typically project-specific.

```markdown
---
title: "Experiment Log — [Series Name] — [Project Shortname]"
tags:
  - log/experiment
  - model/[model-type]
  - project/[project-shortname]
relatedProjects:
  - "[[projects/project-name]]"
---

## Best Result So Far

| Metric | Value | Run | Date |
|--------|-------|-----|------|
| [Metric 1] | [Value] | [run-id] | YYYY-MM-DD |
| [Metric 2] | [Value] | [run-id] | YYYY-MM-DD |

## Runs

### [run-id] — YYYY-MM-DD
**Config:** [model architecture, batch size, learning rate, epochs, dataset]
**Changes:** [What's different from previous run]
**Result:** [Key metrics, one line]
**Notes:** [Observations, issues, ideas for next run]
**Tracking:** [Wandb/MLflow link if applicable]
```

---

## Meeting Note

File location: `meetings/YYYY-MM-DD-MEETING [Meeting type].md`

```markdown
---
title: "[Meeting Type] — YYYY-MM-DD"
tags:
  - meeting
  - meeting/[type]        # pi, collaborator, lab, conference
contexts:
  - meeting
date: YYYY-MM-DD
attendees:
  - [Name 1]
  - [Name 2]
relatedProjects:
  - "[[projects/project-1]]"
actionItems:
  - "[[tasks/Action Item 1]]"
  - "[[tasks/Action Item 2]]"
---

## Discussion

[Key topics discussed, organized by theme or agenda item]

## Decisions

- [Decision 1 and rationale]
- [Decision 2 and rationale]

## Action Items

- [ ] [Action 1] → [[tasks/Task Name]] (assigned to [person], due [date])
- [ ] [Action 2] → [[tasks/Task Name]] (assigned to [person], due [date])

## Notes

[Anything else worth recording]
```
