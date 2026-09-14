---
name: obsidian-vault
description: Maintain and retrieve a human-and-agent shared software-development knowledge base in an Obsidian vault. Use for project architecture, codebase maps, technical decisions, dev logs, project todos, priorities, blockers, work progress, codebase relearning, and on-demand daily or weekly reports. Use after significant development work when durable project knowledge or task state changed. This skill does not replace coding, planning, testing, debugging, or review workflows; the repository remains the source of truth for code behavior.
license: MIT
compatibility: Requires filesystem read/write access to an Obsidian or Markdown vault. Git and ripgrep are optional but recommended.
---

# Obsidian Development Vault

Use an Obsidian vault as a durable, human-readable workspace jointly maintained by the user and coding agents.

This skill manages **project knowledge and work state**, not the software-development methodology itself. Other skills may decide how to plan, implement, test, debug, or review code.

## Core model

Treat the vault as a shared development operating layer:

```text
Global Todo.md                  What projects are active, paused, or completed?
Projects/<Project>/
  <Project> Index.md            Project entry point + project priority
  Todo.md                       What work remains? What is blocked? What is done?
  Architecture.md               How does the system work now?
  Codebase Map.md               Where does the implementation live?
  Decisions/                    Why did important technical choices change?
  Dev Logs/                     What happened over time?
  Tasks/                        Optional deep context for complex tasks only
```

The user and agents may both edit these files directly.

## Core principles

1. **The repository is the source of truth for code behavior.**
   Vault notes explain architecture, intent, history, decisions, progress, and useful code landmarks. If notes disagree with the repository, verify against the repository before correcting durable technical knowledge.

2. **The vault is co-owned by human and agents.**
   Preserve human wording and intent. Do not rewrite, delete, reprioritize, or close work merely to make the notes look cleaner.

3. **Separate current truth from history.**
   `Architecture.md`, `Codebase Map.md`, and `Todo.md` describe current state. Decision notes and dev logs preserve how and why that state evolved.

4. **Retrieve narrowly.**
   Never load the whole vault. Start from `Global Todo.md` or the project index, then read only the notes relevant to the request.

5. **Prefer durable knowledge over transcripts.**
   Preserve architecture changes, meaningful decisions, important debugging discoveries, blockers, and significant progress. Do not record every command or conversational turn.

6. **Reports are views, not source data.**
   Do not automatically create daily, weekly, todo, or status reports. Generate them in chat only when the user asks, unless the user explicitly asks to save a report.

7. **Preserve existing vault conventions.**
   Inspect existing folders, filenames, properties/frontmatter, tags, and wikilink conventions before introducing defaults from this skill.

8. **Never store secrets.**
   Do not write credentials, API keys, tokens, passwords, private keys, or similar sensitive authentication material into the vault.

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
- work becomes an objective blocker for other active work;
- a production, reliability, security, or data-loss issue makes urgency clear;
- the implementation dependency order makes a priority adjustment unambiguous.

Do not silently make strategic reprioritization based only on preference. When a major change is only a recommendation, present it as a recommendation rather than changing the vault.

## Resolve the vault

Resolve the vault location in this order:

1. `OBSIDIAN_VAULT` environment variable.
2. `~/.config/agent-obsidian/vault-path` containing one path.
3. Ask the user for the vault path if neither exists.

Validate the path before writing. Prefer an Obsidian vault containing `.obsidian/`, but allow a normal Markdown directory when the user intentionally uses one.

Do not recursively scan the user's entire home directory looking for a vault.

## Resolve the current project

Inside a Git repository, identify the root with:

```bash
git rev-parse --show-toplevel
```

Use the repository root basename as the default project name, but prefer an existing matching project index.

When not inside a repository, resolve the project from the user's request or from an existing project index.

If multiple projects genuinely match, avoid writing until the target project is clear.

## Default vault layout

Follow an existing vault organization when one exists. Otherwise use:

```text
Global Todo.md
Projects/
  <Project>/
    <Project> Index.md
    Todo.md
    Architecture.md
    Codebase Map.md
    Decisions/
    Dev Logs/
    Tasks/
```

Use `references/note-templates.md` when creating new files and no established template exists.

## Global Todo

`Global Todo.md` is the human-friendly project dashboard.

It must contain **projects only**, not the task list for every project.

Recommended sections:

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

### Global Todo rules

- Keep it small and quickly scannable.
- Project status belongs to one of: `active`, `paused`, `completed`.
- Active/paused project entries should display their current priority.
- Do not duplicate project tasks here.
- When a project is created, ensure it is represented here.
- When a project status or priority changes, synchronize this dashboard with the project index.

The **project index property is canonical** for project status and priority. `Global Todo.md` is the editable dashboard/mirror.

Because humans may edit the dashboard directly, if an agent detects a mismatch caused by an apparent human edit, preserve the human change and synchronize the project index when intent is clear. If the source of the conflict cannot be determined and it materially changes priorities, surface the mismatch instead of silently discarding either value.

## Project Index

`<Project> Index.md` is the project entry point and canonical source for project-level status and priority.

Recommended properties:

```yaml
---
type: project
project: <Project>
status: active
priority: P1
updated: YYYY-MM-DD
---
```

Keep the body compact and link prominently to:

- `[[Todo]]`
- `[[Architecture]]`
- `[[Codebase Map]]`
- important decisions or knowledge when useful

`Todo` should be easy for a human to find from the project index.

## Project Todo

Every active project should have exactly one obvious `Todo.md` as the canonical task/status surface.

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

### Todo rules

- `Todo.md` is canonical for normal task **status and priority**.
- Humans and agents may both edit it.
- Preserve user-authored task wording unless changing it is necessary for correctness.
- Agents may add discovered subtasks, blockers, implementation context, and objectively completed subtasks.
- Agents may move a task to `In Progress` when actually beginning it.
- Do not mark major work complete merely because code was written. Verify acceptance criteria, tests, repository state, or explicit user confirmation.
- Do not delete tasks merely because they appear stale. Move or close them only when the state is supported.
- Do not invent deadlines.
- Do not automatically sort/rewrite the whole file after a small change.
- Keep `Completed` useful; archive old completion detail into dev logs when the section becomes noisy.

### Simple vs complex tasks

Most tasks should remain simple Markdown checkboxes in `Todo.md`.

Create `Tasks/<Task Title>.md` only when a task needs substantial context such as:

- acceptance criteria;
- investigation notes;
- design decisions;
- many subtasks;
- significant blockers;
- detailed code references.

Link it from `Todo.md`:

```markdown
- [ ] [P1] [[Tasks/Automatic Calibration]]
```

For a linked complex task, keep the displayed priority/status in `Todo.md` synchronized with the task note properties. `Todo.md` remains the primary human-facing task-state surface.

If a human edits one surface and the two disagree, preserve the apparent human change and synchronize the other when intent is clear. Do not silently overwrite an ambiguous conflict.

## Architecture

`Architecture.md` describes the **current system architecture**.

Update it when durable system-level design changes, including:

- component boundaries;
- data flow;
- deployment topology;
- storage or database responsibilities;
- queues/workers/background processing;
- major dependencies;
- important constraints and tradeoffs.

Include links to relevant decisions and useful code references.

Do not create architecture churn for trivial implementation details.

## Codebase Map

`Codebase Map.md` helps humans and agents quickly relearn and navigate the repository.

Record only useful landmarks:

- entry points;
- major modules and responsibilities;
- important files/directories;
- key types/functions/symbols;
- relationships between subsystems;
- non-obvious implementation locations.

Example:

```text
src/scanner/controller.rs — scan lifecycle orchestration
src/scanner/transport.rs — film transport control
ScanController — main scan coordinator
FilmTransport — transport abstraction
```

Do not document every file. The goal is a fast mental map into the real codebase.

## Decision Notes

Create a decision note when a meaningful technical choice changes or constrains the system.

Default naming:

```text
Decisions/YYYY-MM-DD <Decision Title>.md
```

Capture:

- context;
- decision;
- why;
- alternatives considered when relevant;
- consequences/tradeoffs;
- affected architecture/code;
- related notes.

Decision notes are historical. Never rewrite an old decision to make history look cleaner. If it is reversed, create a new decision note and link the old and new decisions.

## Dev Logs

Dev logs preserve meaningful chronological work and are distinct from reports.

Default naming:

```text
Dev Logs/YYYY-MM-DD.md
```

Record useful development history such as:

- what was worked on;
- significant changes;
- important discoveries;
- technical decisions;
- blockers/problems;
- meaningful validation/tests;
- next steps when known.

A dev log is not a transcript and should not contain every command.

When multiple significant work sessions happen on one day, update the same daily note unless the user's existing convention says otherwise.

## Retrieve project knowledge

When asked about architecture, history, work status, unfinished work, a past decision, or how to relearn a project:

1. Identify the project.
2. Read its project index.
3. Read only the smallest relevant current-state file(s).
4. Search titles/content for the specific topic.
5. Follow relevant wikilinks/backlinks.
6. For questions about current code behavior, inspect the repository before treating vault notes as authoritative.

Prefer fast search tools such as `rg` when available:

```bash
rg -n -i "auth|session|jwt" "$OBSIDIAN_VAULT/Projects/<Project>"
rg -n -F "[[Backend Architecture]]" "$OBSIDIAN_VAULT/Projects/<Project>"
```

## Update after development work

After **significant** implementation, debugging, refactoring, or architectural work, preserve durable changes when this skill is active or relevant.

Use this order:

1. Verify what actually changed from repository state and tests.
2. Update today's dev log with a concise account of meaningful work.
3. Update `Todo.md` only where task progress, blockers, or newly discovered work actually changed.
4. If current architecture changed, update `Architecture.md`.
5. If codebase landmarks changed materially, update `Codebase Map.md`.
6. If a meaningful technical decision was made, create a decision note.
7. Update the project index only when project metadata or navigation changed.
8. Synchronize project priority/status with `Global Todo.md` when those values changed.

Do not generate a daily/weekly report as part of this workflow.

## Todo and progress queries

When the user asks questions such as:

- "What's the todo list for FilmEnlarger?"
- "What's left on this project?"
- "What's blocked?"
- "What should I work on next?"
- "What are my highest-priority tasks?"

Read `Todo.md` first. Read dev logs only when they are needed to clarify ambiguous progress.

For "what should I work on next?", consider task priority, blockers, current in-progress work, and explicit dependency relationships. Do not invent a strategic priority that is absent from the vault; label additional suggestions as recommendations.

## Cross-project queries

When the user asks:

- "What projects am I working on?"
- "What's my highest-priority project?"
- "What are the top tasks across my active projects?"

Start from `Global Todo.md`.

For project-only questions, do not read every project's `Todo.md`.

For cross-project task questions, identify the relevant active/high-priority projects from `Global Todo.md`, then read only their `Todo.md` files as needed.

## On-demand reports

Only generate a report when the user asks for one.

Examples:

- daily development summary;
- weekly engineering report;
- project progress report;
- architecture-change summary;
- blockers report.

For a date-range report:

1. Determine the requested date range.
2. Read dev logs within that range.
3. Read the current `Todo.md` for present state.
4. Read decision notes from that range when relevant.
5. Inspect architecture/codebase notes if the report asks about system changes.
6. Verify important completion claims against the repository when practical.
7. Answer in chat unless the user explicitly asks to save the report.

A useful engineering report may include:

- Completed
- In Progress
- Decisions / Architecture Changes
- Problems / Blockers
- Remaining Work
- Recommended Next Focus

Do not treat a task as completed merely because a dev log mentioned it.

## Code review and relearning workflow

When helping the user review or relearn a codebase:

1. Read the project index.
2. Read `Architecture.md` and relevant parts of `Codebase Map.md`.
3. Read `Todo.md` if current work state matters.
4. Read relevant decision notes that explain unusual design choices.
5. Inspect the corresponding repository files and symbols.
6. Explain the current code using vault history as context, not as a substitute for code inspection.
7. Update stale durable notes only after verifying the repository.

## Project initialization

When asked to initialize a project in the vault:

1. Inspect the repository enough to understand its actual structure.
2. Inspect existing vault conventions.
3. Create the project folder only if it does not exist.
4. Create the minimal files:
   - `<Project> Index.md`
   - `Todo.md`
   - `Architecture.md`
   - `Codebase Map.md`
5. Add the project to `Global Todo.md` with status and priority.
6. Do not fabricate a detailed task backlog. Seed todos only from explicit user input, repository evidence, or clearly existing work.
7. Create `Decisions/`, `Dev Logs/`, or `Tasks/` as needed rather than filling them with empty placeholder notes.

## Boundaries

This skill does **not**:

- replace Git history;
- replace the repository as source of truth;
- replace a team issue tracker or project-management system;
- automatically synchronize to Notion or another service;
- automatically generate reports;
- replace coding/planning/TDD/debugging/review skills;
- require semantic/vector indexing;
- store secrets.
