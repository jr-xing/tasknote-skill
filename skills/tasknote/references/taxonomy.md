# Tag Taxonomy and Context Values

## Contexts

Contexts describe the type of work being done. Use in the `contexts` frontmatter field with `@` prefix in mtn create commands.

| Context | mtn syntax | Use when |
|---------|-----------|----------|
| data | `@data` | Data acquisition, cleaning, preprocessing, format conversion, annotation, QC |
| experiment | `@experiment` | Model training, hyperparameter tuning, evaluation, benchmarking |
| code | `@code` | Software development, pipeline building, debugging, refactoring |
| writing | `@writing` | Paper drafting, grant writing, documentation, manuscripts |
| plan | `@plan` | Planning, design decisions, architecture, scoping, roadmaps |
| admin | `@admin` | Email, access requests, IRB, paperwork, logistics |
| review | `@review` | Code review, paper review, PR review, peer feedback |
| meeting | `@meeting` | Meeting-related tasks and notes |

A task should have 1-2 contexts maximum. Pick the primary activity type.

## Tags

Hierarchical tags for filtering and grouping. Use `#tag` syntax in mtn create, and `tag/subtag` for hierarchy.

### Project Tags

Format: `project/[year-month-shortname]`

Examples:
- `project/2026-02-MICCAI`
- `project/2026-03-AHA-Fellowship`
- `project/2026-01-FoundationModel`

These mirror the project folder names and enable quick filtering in Obsidian. Every task should have its project tag in addition to the `projects` frontmatter link.

### Modality Tags

Format: `modality/[type]`

| Tag | Imaging modality |
|-----|-----------------|
| `modality/cine` | Cardiac cine MRI |
| `modality/lge` | Late gadolinium enhancement |
| `modality/t1` | T1 mapping |
| `modality/t2` | T2 mapping |
| `modality/ct` | CT imaging |
| `modality/echo` | Echocardiography |

### Source Tags

Format: `source/[institution-or-dataset]`

| Tag | Data source |
|-----|------------|
| `source/ukbb` | UK Biobank |
| `source/yale` | Yale University |
| `source/upenn` | University of Pennsylvania |
| `source/internal` | Internal/lab data |

### Model Tags

Format: `model/[type]`

| Tag | Model type |
|-----|-----------|
| `model/segmentation` | Segmentation models |
| `model/classification` | Classification models |
| `model/detection` | Detection models |
| `model/foundation` | Foundation models |
| `model/registration` | Registration models |

### Venue Tags

Format: `venue/[name]`

Examples: `venue/miccai`, `venue/aha`, `venue/tmi`, `venue/media`

### Log Tags

| Tag | Note type |
|-----|-----------|
| `log/data` | Data log notes |
| `log/experiment` | Experiment log notes |

### Card Tags

| Tag | Card type |
|-----|-----------|
| `card/detail` | Detail cards (task-specific supplementary content) |
| `card/reference` | Reference cards (shared permanent knowledge) |

### Meeting Tags

| Tag | Meeting type |
|-----|-------------|
| `meeting` | Any meeting note |
| `meeting/pi` | PI meeting |
| `meeting/collaborator` | Collaborator meeting |
| `meeting/lab` | Lab meeting |
| `meeting/conference` | Conference-related |

## Inference Rules

When the user mentions certain keywords, infer these tags and contexts:

| User mentions | Infer context | Infer tags |
|--------------|--------------|------------|
| "download", "clean", "preprocess", "convert", "annotate", "QC" | `@data` | — |
| "train", "tune", "epoch", "loss", "Dice", "metrics", "evaluate" | `@experiment` | — |
| "code", "debug", "pipeline", "refactor", "PR", "commit" | `@code` | — |
| "draft", "write", "paper", "manuscript", "abstract", "revision" | `@writing` | — |
| "cine", "CMR cine" | — | `#modality/cine` |
| "LGE", "late gadolinium" | — | `#modality/lge` |
| "T1 map", "T1 mapping" | — | `#modality/t1` |
| "T2 map", "T2 mapping" | — | `#modality/t2` |
| "UK Biobank", "UKBB", "Biobank" | — | `#source/ukbb` |
| "Yale" | — | `#source/yale` |
| "UPenn", "Penn" | — | `#source/upenn` |
| "segmentation", "segment" | — | `#model/segmentation` |
| "MICCAI" | — | `#project/2026-02-MICCAI`, `#venue/miccai` |
