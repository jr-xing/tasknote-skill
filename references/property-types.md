# Property Types Reference

Authoritative reference for frontmatter property names, types, and formats. When in doubt, prefer ISO-style values.

## Quick Reference

| Property | Type | Example |
|----------|------|---------|
| title | text | `"My Task"` |
| status | text | `"open"`, `"in-progress"`, `"done"` |
| priority | text | `"low"`, `"normal"`, `"high"` |
| due | text (date) | `"2025-01-15"` |
| scheduled | text (date) | `"2025-01-10"` |
| completedDate | text (date) | `"2025-01-20"` |
| dateCreated | text (datetime) | `"2025-01-01T08:00:00Z"` |
| dateModified | text (datetime) | `"2025-01-15T10:30:00Z"` |
| tags | list | `["work", "urgent"]` |
| contexts | list | `["office", "computer"]` |
| projects | list | `["[[Project A]]"]` |
| timeEstimate | number | `120` (minutes) |
| recurrence | text | `"FREQ=WEEKLY;BYDAY=MO"` |
| recurrenceAnchor | text | `"scheduled"` or `"completion"` |
| timeEntries | list (objects) | See [Time Entries](#time-entries) |
| blockedBy | list (objects) | See [Dependencies](#dependencies) |
| reminders | list (objects) | See [Reminders](#reminders) |
| complete_instances | list | `["2025-01-08", "2025-01-15"]` |
| skipped_instances | list | `["2025-01-22"]` |

## Date Properties

All dates are **text strings** in frontmatter. Formats:

- **Date only:** `YYYY-MM-DD` (for `due`, `scheduled`, `completedDate`)
- **Datetime:** ISO 8601 with timezone (for `dateCreated`, `dateModified`)

```yaml
due: "2025-01-15"
scheduled: "2025-01-10T09:00:00"
dateCreated: "2025-01-01T08:00:00Z"
```

## List Properties

Lists must be arrays, even for single values:

```yaml
tags:
  - work
  - documentation
contexts:
  - office
projects:
  - "[[Website Redesign]]"
```

## Complex Properties

### Time Entries

Field names are `startTime` and `endTime` (NOT `start`/`end`).

```yaml
timeEntries:
  - startTime: "2025-01-15T10:30:00.000Z"    # Required: ISO 8601
    endTime: "2025-01-15T11:15:00.000Z"       # Optional: ISO 8601
    description: "Initial work"               # Optional: text
```

### Dependencies

```yaml
blockedBy:
  - uid: "path/to/blocking-task.md"    # Required: path or wikilink
    reltype: "FINISHTOSTART"           # Required: relationship type
    gap: "P1D"                         # Optional: ISO 8601 duration
```

Relationship types: `FINISHTOSTART`, `STARTTOSTART`, `FINISHTOFINISH`, `STARTTOFINISH`

Current behavior: Dependencies use `FINISHTOSTART`. Blocking evaluation is based on presence/completion state.

Wikilink style also works:

```yaml
blockedBy:
  - uid: "[[tasks/Blocking Task Name]]"
```

### Reminders

**Relative** (before/after due or scheduled):

```yaml
reminders:
  - id: "rem_1"
    type: "relative"
    relatedTo: "due"                   # "due" or "scheduled"
    offset: "-PT1H"                    # ISO 8601 duration
    description: "1 hour before due"
```

**Absolute** (specific time):

```yaml
reminders:
  - id: "rem_2"
    type: "absolute"
    absoluteTime: "2025-01-15T09:00:00Z"
    description: "Morning reminder"
```

### Recurrence

Uses RFC 5545 RRULE format:

```yaml
recurrence: "FREQ=DAILY"
recurrence: "FREQ=WEEKLY;BYDAY=MO,WE,FR"
recurrence: "FREQ=MONTHLY;BYMONTHDAY=1"
recurrenceAnchor: "scheduled"          # or "completion"
```

## Complete Example

```yaml
---
title: "Complete quarterly report"
status: "in-progress"
priority: "high"
due: "2025-01-31"
scheduled: "2025-01-25"
tags:
  - work
  - reports
contexts:
  - office
projects:
  - "[[Q1 Planning]]"
timeEstimate: 240
dateCreated: "2025-01-01T08:00:00Z"
dateModified: "2025-01-20T14:30:00Z"
timeEntries:
  - startTime: "2025-01-20T10:00:00.000Z"
    endTime: "2025-01-20T11:30:00.000Z"
    description: "Draft outline"
blockedBy:
  - uid: "tasks/gather-data.md"
    reltype: "FINISHTOSTART"
reminders:
  - id: "rem_1"
    type: "relative"
    relatedTo: "due"
    offset: "-P1D"
    description: "Due tomorrow"
---
```

## Custom User Fields

Additional properties can be defined with types: text, number, date, boolean, list. See TaskNotes Settings → Task Properties → Custom User Fields.

## Field Mapping

All property names can be customized via Settings → Task Properties → Field Mapping. If changed, TaskNotes reads/writes using custom names.
