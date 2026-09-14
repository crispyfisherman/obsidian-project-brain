# Obsidian Development Vault — Default Note Templates

Use these templates only when the vault does not already have established conventions.

Priority vocabulary:

- `P0` — Critical
- `P1` — High
- `P2` — Normal
- `P3` — Low

## Global Todo

```markdown
# Global Todo

## Active Projects
- [P1] [[Projects/<Project>/<Project> Index|<Project>]]

## Paused

## Completed
```

Keep this file project-only. Never duplicate every project's tasks here.

## Project Index

```markdown
---
type: project
project: <Project>
status: active
priority: P1
updated: YYYY-MM-DD
---

# <Project>

## Current Work
- [[Todo]]

## System
- [[Architecture]]
- [[Codebase Map]]

## Important Decisions
- [[Decisions/YYYY-MM-DD Decision Title]]

## Recent Dev Logs
- [[Dev Logs/YYYY-MM-DD]]
```

## Todo

```markdown
---
type: todo
project: <Project>
updated: YYYY-MM-DD
---

# <Project> Todo

## In Progress
- [ ] [P1] Current task

## Next
- [ ] [P2] Near-term task

## Blocked
- [ ] [P1] Blocked task
  - Blocked by: reason

## Backlog
- [ ] [P3] Future task

## Completed
- [x] [P1] Completed task — YYYY-MM-DD
```

## Architecture

```markdown
---
type: architecture
project: <Project>
updated: YYYY-MM-DD
---

# Architecture

## Overview

## Components

## Data Flow

## External Dependencies

## Important Constraints

## Current Tradeoffs

## Key Code References

## Related Decisions
- [[Decisions/YYYY-MM-DD Decision Title]]
```

## Codebase Map

```markdown
---
type: codebase-map
project: <Project>
updated: YYYY-MM-DD
---

# Codebase Map

## Entry Points

## Major Modules

## Key Files

## Key Symbols

## Important Relationships

## Areas That Need Relearning / Review
```

## Decision

```markdown
---
type: decision
project: <Project>
date: YYYY-MM-DD
status: active
---

# <Decision Title>

## Context

## Decision

## Why

## Alternatives Considered

## Consequences / Tradeoffs

## Affected Code

## Related
- [[Architecture]]
- [[Codebase Map]]
```

## Dev Log

```markdown
---
type: dev-log
project: <Project>
date: YYYY-MM-DD
---

# YYYY-MM-DD

## Worked On

## Changed

## Decisions / Discoveries

## Problems / Blockers

## Validation

## Next

## Related
- [[Todo]]
```

## Complex Task

Use a separate task note only when the task needs substantial context.

```markdown
---
type: task
project: <Project>
status: in-progress
priority: P1
created: YYYY-MM-DD
updated: YYYY-MM-DD
---

# <Task Title>

## Goal

## Acceptance Criteria
- [ ]

## Progress

## Blockers

## Notes

## Code References

## Related
- [[Todo]]
```

Mirror this task in `Todo.md`, for example:

```markdown
- [ ] [P1] [[Tasks/<Task Title>]]
```

`Todo.md` is the primary human-facing task-state surface; keep the linked task note metadata synchronized.
