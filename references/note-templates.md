# Obsidian Project Brain — Default Note Templates

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
repository: github.com/owner/repo
updated: YYYY-MM-DD
---

# <Project>

## Current Work

[[Todo]]

## Repository

- Repository: `github.com/owner/repo`
- Domain context: `CONTEXT.md`
- Architecture decisions: `docs/adr/`
- Specs / plans: `specs/`

## Recent Work

- [[Dev Logs/YYYY-MM-DD]]

## Notes

- [[Notes/Useful Mental Model]]
```

Remove repo pointers that do not actually exist. Do not copy their contents into Obsidian.

## Todo

```markdown
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

## Dev Log

```markdown
---
type: dev-log
project: <Project>
repository: github.com/owner/repo
date: YYYY-MM-DD
---

# YYYY-MM-DD <Project> Dev Log

Project: [[<Project> Index]]
Repository: `github.com/owner/repo`

## Worked On

- 

## Repository Knowledge

- `CONTEXT.md` / ADR / spec / plan / PR / commit references when relevant

## Progress

- 

## Blockers

- 

## Next

- See [[Todo]]
```

Keep this concise. Reference canonical repo artifacts instead of copying them.

## Personal / Learning Note

```markdown
---
type: note
project: <Project>
repository: github.com/owner/repo
updated: YYYY-MM-DD
---

# Note Title

Project: [[<Project> Index]]

## Why I Need To Remember This


## Mental Model / Learning


## Repo References

- `path/to/file`
- `CONTEXT.md`
- `docs/adr/...`
```

Do not use a personal note to duplicate canonical repository documentation.

## Complex Task Note

Use a separate task note only when the task needs substantial context.

```markdown
---
type: task
project: <Project>
status: in-progress
priority: P1
repository: github.com/owner/repo
updated: YYYY-MM-DD
---

# <Task Title>

Project: [[<Project> Index]]
Todo: [[Todo]]

## Goal


## Acceptance Criteria

- [ ]

## Context


## Blockers


## Repo References

- `path/to/file`
- `CONTEXT.md`
- `docs/adr/...`
```

Mirror this task in `Todo.md`, for example:

```markdown
- [ ] [P1] [[Tasks/<Task Title>]]
```

`Todo.md` remains the primary human-facing task-state surface. Keep linked task metadata synchronized without silently overriding ambiguous human edits.
