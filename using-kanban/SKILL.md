---
name: using-kanban
description: Use when planning work items, managing backlogs, creating epics/features/stories/tasks, or organizing development work in Agile/SCRUM methodology
---

# Using Kanban

## Overview

Kanban is an Agile/SCRUM work item management system using a hierarchical folder structure with markdown files. Each work item has YAML frontmatter for metadata and markdown content for details.

**Core principle:** Break large initiatives into smaller deliverable units, track state through ToDo → InProgress → InReview → Done.

## When to Use

- Planning new development initiatives
- Creating work items (epics, features, stories, bugfixes, tasks)
- Starting work from the backlog
- Tracking progress across multiple items
- Organizing work into sprints or iterations

**Not for:** Ad-hoc fixes (use `/fix-errors`), quick tasks without tracking needs.

## Folder Structure

```
<PROJECT_ROOT>/.kanban/
├── epics/
│   └── epic-<id>-<slug>/
│       ├── _epic.md
│       └── features/
│           └── feature-<id>-<slug>/
│               ├── _feature.md
│               └── items/
│                   ├── story-<id>.md
│                   ├── bugfix-<id>.md
│                   └── task-<id>.md
└── standalone/
    ├── stories/
    ├── bugfixes/
    └── tasks/
```

**CRITICAL:** Kanban files go in the **project root**, not in the workflow files location.

## Work Item Types

| Type | Description | Parent | Workflow |
|------|-------------|--------|----------|
| **Epic** | Large initiative spanning features | None | `/create-epic` |
| **Feature** | Significant functionality | Epic (optional) | `/create-feature` |
| **Story** | User story with acceptance criteria | Feature/Epic | `/create-story` |
| **Bugfix** | Planned fix with root cause analysis | Feature/Story | `/create-bugfix` |
| **Task** | Small technical work (< 1 day) | Story/Feature | `/create-task` |

## States

| State | Meaning |
|-------|---------|
| **ToDo** | Ready to begin |
| **InProgress** | Being worked on |
| **InReview** | Awaiting review |
| **Done** | Completed |
| **Blocked** | Cannot proceed (requires `blocked_reason`) |

## File Format

Each item uses YAML frontmatter + markdown:

```yaml
---
id: story-001
title: User can log in with email
type: story
parent: feature-001
state: ToDo
priority: High
effort: 3
tags: [auth, security]
assignee: ""
created: 2026-01-15
updated: 2026-01-15
acceptance_criteria:
  - User can enter email and password
  - Invalid credentials show error message
  - Successful login redirects to dashboard
---

# User can log in with email

## Description
As a registered user, I want to log in with my email and password, so that I can access my account.

## Acceptance Criteria
- [ ] User can enter email and password
- [ ] Invalid credentials show error message
- [ ] Successful login redirects to dashboard

## Technical Notes
Use existing auth service. Follow OAuth 2.0 patterns.
```

## Quick Input Shortcuts

When prompted, use these shortcuts:

| Shortcut | Field | Example |
|----------|-------|---------|
| `P:` | Purpose/Description | `P: Enable user login` |
| `A:` | Acceptance Criteria | `A: User can enter email` |
| `V:` | Business Value | `V: Reduce support tickets` |
| `T:` | Tags | `T: security, auth` |
| `PR:` | Priority | `PR: High` |
| `E:` | Effort/Points | `E: 5` |
| `D:` | Dependencies | `D: feature-001` |
| `GEN` | Auto-generate | LLM generates based on context |

## Workflow Integration

### Starting Work
```
/start-kanban-work              # List all incomplete items
/start-kanban-work features     # List incomplete features
/start-kanban-work todo         # List items in ToDo state
/start-kanban-work feature-001  # Start specific item
```

### Creating Items
- `/create-epic` - Large initiatives with 1-3-1 decomposition
- `/create-feature` - Features with story/task generation
- `/create-story` - User stories with acceptance criteria
- `/create-bugfix` - Planned fixes with root cause
- `/create-task` - Small technical tasks

## 1-3-1 Decision Method

Used in epics and features to structure decisions:

1. **1 Core Problem** - Clear statement of what to solve
2. **3 Options** - Quick/Simple, Balanced, Comprehensive approaches
3. **1 Recommendation** - Best approach with reasoning

## ID Generation

**CRITICAL: Use the ID counter file to ensure unique numbering.**

### Counter File: `.kanban/_counters.yaml`

This file tracks the last used ID for each work item type:

```yaml
# .kanban/_counters.yaml
# Last assigned ID numbers - DO NOT manually edit
epic: 3
feature: 12
story: 45
bugfix: 8
task: 67
```

### Process

1. **Read** `.kanban/_counters.yaml` (create if missing with all zeros)
2. **Increment** the counter for the item type
3. **Generate ID** using format `<type>-<number>` (zero-padded to 3 digits)
4. **Update** the counter file with new value
5. **Create** the work item with the new ID

**ID format:** `<type>-<number>` (e.g., `epic-001`, `feature-012`, `story-046`)

### Initialize Counter File

If `.kanban/_counters.yaml` doesn't exist, create it:

```yaml
# .kanban/_counters.yaml
# Last assigned ID numbers - DO NOT manually edit
epic: 0
feature: 0
story: 0
bugfix: 0
task: 0
```

## Best Practices

1. **Use `_counters.yaml`** to get next ID (never manually assign)
2. **Start with Epic** for large initiatives
3. **Break down** epics into features before implementation
4. **Keep tasks small** (< 1 day)
5. **Document acceptance criteria** for stories/bugfixes
6. **Update states** to reflect actual progress
7. **Use tags** for cross-cutting concerns (security, performance)

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| **Reusing existing IDs** | Always use `_counters.yaml` to get next ID |
| Creating files in wrong location | Always use `<PROJECT_ROOT>/.kanban/` |
| Skipping acceptance criteria | Define criteria before implementation |
| Features too large | Decompose into 3-7 stories |
| Not updating state | Change state when starting/finishing work |
| Creating standalone when epic exists | Nest features under parent epic |

## State Transitions

```
ToDo → InProgress   # Start working
InProgress → InReview  # Submit for review
InReview → Done     # Approved
InProgress → Blocked   # Cannot proceed
Blocked → InProgress   # Blocker resolved
```

When blocking, always add `blocked_reason`:
```yaml
state: Blocked
blocked_reason: "Waiting for API access credentials"
```
