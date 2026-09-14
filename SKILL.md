---
name: obsidian-project-brain
description: Maintain a human-and-agent shared project cockpit in Obsidian for cross-project priorities, per-project todos, development logs, progress history, and personal learning notes. Use when the user asks to initialize or update project tracking, record a development session, review current or remaining work, recall recent progress, or generate an on-demand project or weekly report. The Git repository remains authoritative for engineering knowledge such as CONTEXT.md, ADRs, specs, architecture, code, and tests.
license: MIT
compatibility: Requires filesystem read/write access to an Obsidian or Markdown vault. Git and ripgrep are optional but recommended.
---

# Obsidian Project Brain

Use an Obsidian vault as a **human + agent shared development cockpit**.

This skill tracks what the user is working on, what remains, what happened over time, and useful personal/project notes. It does **not** duplicate the repository's canonical engineering documentation.

## Ownership model

```text
Obsidian Project Brain                 Git repository
----------------------                 --------------
Global project dashboard               Source code
Project priority/status                CONTEXT.md / CONTEXT-MAP.md
Per-project Todo.md                    ADRs / architecture docs
Development journal                    Specs / plans / tickets
Progress history                       Tests
Personal learning / mental notes       Canonical engineering decisions
Pointers to repo artifacts             Implementation truth
```

When durable engineering knowledge already exists in the repository, point to it from Obsidian instead of copying it.

Examples:

- Architectural decisions belong in repo ADRs. A dev log may record that `docs/adr/0008-worker-architecture.md` was created or changed.
- Domain vocabulary belongs in `CONTEXT.md` or `CONTEXT-MAP.md` when the repo uses those files.
- Specs, plans, tickets, and design docs created by other engineering skills stay in the repository.
- Obsidian records progress, current work, personal context, and where the canonical repo knowledge lives.

## Core model

```text
Global Todo.md
Projects/<Project>/
  <Project> Index.md
  Todo.md
  Dev Logs/
  Notes/                optional personal/project learning notes
  Tasks/                optional deep context for complex tasks
```

The user and agents may both edit these files directly.

## Core principles

1. **The Git repository is authoritative for engineering truth.**
   Code, tests, `CONTEXT.md`, ADRs, architecture docs, specs, and repo-local plans outrank Obsidian notes when they disagree.

2. **Obsidian is the user's development cockpit, not a second engineering source of truth.**
   Track projects, priorities, todos, progress, journal history, personal learning, and pointers to canonical repo artifacts.

3. **The vault is co-owned by human and agents.**
   Preserve human wording and intent. Do not rewrite, delete, reprioritize, or close work merely to make notes cleaner.

4. **Repository binding must be explicit.**
   A repo-backed project should identify its repository. Never write a development log until the current checkout resolves to exactly one Obsidian project.

5. **Reports are views, not source data.**
   Do not automatically create daily, weekly, todo, or status reports. Generate them only when the user asks, and keep them in chat unless the user explicitly asks to save them.

6. **Logging is deliberate.**
   Do not automatically log every discussion, skill invocation, command, or coding step. Record a session when the user asks to log, update, or preserve the work, or explicitly invokes this skill for that purpose.

7. **Prefer concise durable history over transcripts.**
   Dev logs capture meaningful work, outcomes, blockers, repo artifacts created or changed, and next steps. Never save raw chat transcripts by default.

8. **Retrieve narrowly.**
   Start from the project index or `Global Todo.md`, then read only the files needed for the user's question.

9. **Preserve existing vault conventions.**
   Follow existing folder, filename, frontmatter, tag, and wikilink conventions when present.

10. **Never store secrets.**
    Do not write credentials, API keys, tokens, passwords, private keys, or similar authentication material into the vault.

## Priority model

Use exactly four priority levels for projects and tasks:

```text
P0 = Critical — immediate, blocking, production/reliability emergency, or explicitly top priority
P1 = High     — important active work
P2 = Normal   — should be done, but not urgent
P3 = Low      — optional, exploratory, or future work
```

Priority is shared mutable state. Humans and agents may change it.

Agents may change priority when:

- the user explicitly requests it;
- a task becomes an objective blocker for other active work;
- a production, reliability, security, or data-loss issue makes urgency clear;
- implementation dependency order makes the adjustment unambiguous.

Do not silently make strategic reprioritization based only on preference. If a change is only a recommendation, present it as a recommendation instead of mutating the vault.

## Resolve the vault

Resolve the vault location in this order:

1. `OBSIDIAN_VAULT` environment variable.
2. `~/.config/agent-obsidian/vault-path` containing one path.
3. Ask the user for the vault path if neither exists.

Validate the path before writing. Prefer an Obsidian vault containing `.obsidian/`, but allow a normal Markdown directory when intentionally used as the vault.

Do not recursively scan the entire home directory looking for a vault.

## Resolve repository identity

Inside a Git repository, determine the checkout root:

```bash
git rev-parse --show-toplevel
```

Then inspect the primary remote when available:

```bash
git remote get-url origin
```

Normalize common SSH and HTTPS forms to a stable identity. For example:

```text
git@github.com:owner/repo.git
https://github.com/owner/repo.git
https://github.com/owner/repo
```

all represent:

```text
github.com/owner/repo
```

Prefer the normalized remote identity over a local filesystem path because local paths can change across machines.

If there is no remote, use repository root basename plus the local Git root as a fallback mapping. If that could match multiple projects, do not write until the target is clear.

## Project binding

Every repo-backed project index should include repository identity.

Recommended frontmatter:

```yaml
---
type: project
project: FilmEnlarger
status: active
priority: P1
repository: github.com/example/FilmEnlarger
updated: YYYY-MM-DD
---
```

`repository` is the canonical binding key.

Before writing a dev log from a repository session:

1. Resolve the current normalized repository identity.
2. Find the project index with the same `repository` value.
3. If exactly one project matches, use it.
4. If none matches and the user asked to initialize the project, create the project mapping.
5. If none matches during an ordinary log/update request, surface that the repo has not been bound yet.
6. If multiple projects match, do not write until the ambiguity is resolved.

## Default vault layout

Follow an existing vault organization when one exists. Otherwise use:

```text
Global Todo.md
Projects/
  <Project>/
    <Project> Index.md
    Todo.md
    Dev Logs/
    Notes/
    Tasks/              optional
```

Use `references/note-templates.md` when creating new files and no established template exists.

## Global Todo

`Global Todo.md` is the cross-project dashboard. It contains **projects only**, not each project's task list.

Recommended structure:

```markdown
# Global Todo

## Active Projects
- [P0] [[Projects/Project A/Project A Index|Project A]]
- [P1] [[Projects/Project B/Project B Index|Project B]]

## Paused
- [P3] [[Projects/Project C/Project C Index|Project C]]

## Completed
- [[Projects/Project D/Project D Index|Project D]]
```

Rules:

- Keep it small and quickly scannable.
- Project status is one of `active`, `paused`, `completed`.
- Active and paused entries display current priority.
- Never duplicate per-project tasks here.
- When project status or priority changes, synchronize the dashboard and project index.
- Project index frontmatter is the structured canonical metadata; `Global Todo.md` is the human-friendly editable dashboard.
- If a human edit creates a mismatch and intent is clear, preserve the human edit and reconcile the other surface. If intent is ambiguous and materially changes priority/status, surface the mismatch instead of guessing.

## Project Index

`<Project> Index.md` is the project entry point.

Recommended frontmatter:

```yaml
---
type: project
project: <Project>
status: active
priority: P1
repository: github.com/owner/repo
updated: YYYY-MM-DD
---
```

Keep the body compact. Make these easy to find:

- `[[Todo]]`
- recent dev logs
- optional `[[Notes/...]]`
- repository identity or location
- pointers to canonical repo knowledge such as `CONTEXT.md`, `CONTEXT-MAP.md`, `docs/adr/`, specs, or plans when they exist

Do not copy repository engineering docs into the project index.

## Project Todo

Every active project should have exactly one obvious `Todo.md` as the canonical human-facing task/status surface.

Recommended structure:

```markdown
# <Project> Todo

## In Progress
- [ ] [P0] Current urgent task
- [ ] [P1] Current task

## Next
- [ ] [P1] Near-term task

## Blocked
- [ ] [P1] Blocked task
  - Blocked by: reason

## Backlog
- [ ] [P2] Normal backlog task
- [ ] [P3] Nice-to-have task

## Completed
- [x] [P1] Meaningful completed task — YYYY-MM-DD
```

Todo rules:

- `Todo.md` is canonical for normal task status and priority.
- Humans and agents may both edit it.
- Preserve user-authored task wording unless changing it is necessary for correctness.
- Agents may add discovered subtasks, blockers, implementation context, and objectively completed subtasks.
- Agents may move a task to `In Progress` when actually beginning it.
- Do not mark major work complete merely because code was written. Verify acceptance criteria, tests, repository state, or explicit user confirmation.
- Do not delete tasks merely because they appear stale.
- Do not invent deadlines.
- Do not automatically sort or rewrite the whole file after a small change.

### Complex task notes

Most tasks should remain checkboxes in `Todo.md`.

Create `Tasks/<Task Title>.md` only when a task needs substantial context such as acceptance criteria, investigation notes, many subtasks, major blockers, or detailed repo references.

Link it from `Todo.md`:

```markdown
- [ ] [P1] [[Tasks/Automatic Calibration]]
```

Keep the displayed task status/priority and linked task metadata synchronized. If a human edit causes a conflict and intent is clear, preserve the human change. Do not silently overwrite an ambiguous conflict.

## Dev Logs

Dev logs preserve meaningful chronological work.

Default naming:

```text
Dev Logs/YYYY-MM-DD.md
```

A dev log should record useful history such as:

- what was worked on;
- meaningful progress;
- blockers/problems;
- validation that matters;
- important discoveries;
- repo artifacts created or changed, such as ADRs, `CONTEXT.md`, specs, PRs, or commits;
- next steps when known.

A dev log is not a transcript and should not duplicate the contents of canonical repository docs.

For example:

```markdown
## Repository Knowledge

- New ADR: `docs/adr/0008-separate-transport-state.md`
- Updated domain context: `CONTEXT.md`
```

When multiple significant sessions happen on one day, update the same daily note unless the user's existing convention says otherwise.

## Personal and learning notes

Use `Notes/` for information that helps the user personally understand or remember a project but does not belong as canonical repository documentation.

Examples:

- a mental model that helps relearn a subsystem;
- personal reminders about confusing code paths;
- learning notes;
- useful links between concepts;
- pointers to important repo files.

Do not use personal notes to create a competing copy of `CONTEXT.md`, ADRs, architecture docs, or specs.

## Retrieve project state

For questions such as:

- "What's the todo list for FilmEnlarger?"
- "What's left on this project?"
- "What's blocked?"
- "What did I work on recently?"
- "What should I pick up next?"

1. Resolve the target project.
2. Read its project index.
3. Read `Todo.md` first for current work state.
4. Read relevant dev logs only when historical progress matters.
5. Follow repo pointers when the question depends on canonical engineering knowledge.
6. Inspect the repository for current code behavior.

## Cross-project queries

For questions such as:

- "What projects am I working on?"
- "What's my highest-priority project?"
- "What are my top tasks across active projects?"

Start from `Global Todo.md`.

For project-only questions, do not read every project's `Todo.md`.

For cross-project task questions, identify relevant active/high-priority projects first, then read only their `Todo.md` files as needed.

## Deliberate logging workflow

When the user asks to log, update, or preserve a development session:

1. Resolve the current repository and its bound Obsidian project.
2. Inspect repository state and recent work enough to avoid inventing progress.
3. Update today's dev log with a concise summary.
4. Update `Todo.md` only where task progress, blockers, newly discovered work, or priority actually changed.
5. Reference repo-local ADRs, `CONTEXT.md`, specs, plans, PRs, or commits that were created or changed.
6. Do not duplicate those repo documents into Obsidian.
7. Update the project index only when metadata, repo pointers, or navigation changed.
8. Synchronize project priority/status with `Global Todo.md` when those values changed.

Do not generate a daily or weekly report as part of logging.

## On-demand reports

Only generate a report when the user asks for one.

Examples:

- daily development summary;
- weekly engineering report;
- project progress report;
- blockers report;
- recent-change summary.

For a date-range report:

1. Determine the requested date range.
2. Read dev logs within that range.
3. Read the current `Todo.md` for present state.
4. Follow referenced repo artifacts when needed to accurately describe engineering changes.
5. Verify important completion claims against the repository when practical.
6. Answer in chat unless the user explicitly asks to save the report.

A useful report may include:

- Completed
- In Progress
- Repository Knowledge / Decisions
- Problems / Blockers
- Remaining Work
- Recommended Next Focus

Do not treat a task as completed merely because a dev log mentioned it.

## Relearning a project

When the user wants to relearn an old codebase:

1. Read the project index and `Todo.md`.
2. Read recent dev logs for historical orientation.
3. Read relevant personal notes if they exist.
4. Follow pointers into repo `CONTEXT.md`, ADRs, specs, or other canonical docs.
5. Inspect the actual repository files and symbols.
6. Explain the current code using the vault as progress/history context, not as a substitute for repository inspection.

## Project initialization

When asked to initialize a repo-backed project:

1. Resolve the repository root and normalized remote identity.
2. Inspect existing vault conventions.
3. Create the project folder only if it does not exist.
4. Create the minimum files:
   - `<Project> Index.md`
   - `Todo.md`
5. Add the project to `Global Todo.md` with status and priority.
6. Add repo pointers such as `CONTEXT.md`, `CONTEXT-MAP.md`, `docs/adr/`, specs, or plans only when they actually exist.
7. Do not fabricate a task backlog.
8. Create `Dev Logs/`, `Notes/`, or `Tasks/` only when needed rather than filling them with placeholders.

## Boundaries

This skill does **not**:

- replace Git history;
- replace the repository as source of engineering truth;
- duplicate repo `CONTEXT.md`, ADRs, architecture docs, specs, or plans;
- replace a team issue tracker or project-management system;
- automatically synchronize to Notion or another service;
- automatically log every discussion or skill invocation;
- automatically generate reports;
- replace coding/planning/TDD/debugging/review skills;
- require semantic/vector indexing;
- store secrets.
