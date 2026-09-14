# obsidian-project-brain

A portable Agent Skill that turns an Obsidian vault into a **human + coding-agent project cockpit**.

It keeps a clean ownership boundary:

```text
Obsidian                              Git repository
--------                              --------------
Project dashboard                     Source code
Project/task priority                 CONTEXT.md / CONTEXT-MAP.md
Todo lists                            ADRs / architecture docs
Development journal                   Specs / plans / tickets
Progress history                      Tests
Personal learning notes               Engineering source of truth
```

The goal is not to replace repo-local engineering documentation. The goal is to give you and your agents a shared place to answer:

- What projects am I working on?
- What is the priority of each project?
- What's left to do in this repo?
- What did I work on this week?
- What's blocked?
- What changed recently?
- Which repo ADR/spec/context file explains that change?
- What should I pick up next?

## Vault model

```text
Obsidian Vault/
├── Global Todo.md
└── Projects/
    └── <Project>/
        ├── <Project> Index.md
        ├── Todo.md
        ├── Dev Logs/
        ├── Notes/          # optional personal/project learning
        └── Tasks/          # optional complex task notes
```

### What each file owns

- `Global Todo.md` — active/paused/completed projects and project priority.
- `<Project> Index.md` — project metadata, repository binding, navigation, and pointers to canonical repo docs.
- `Todo.md` — task state, task priority, blockers, and completion state.
- `Dev Logs/` — concise chronological history of meaningful development work.
- `Notes/` — optional personal mental models, learnings, and reminders that should not become canonical repo documentation.
- `Tasks/` — optional deep context for complex tasks only.

## Repository-first engineering knowledge

This skill assumes your engineering workflow may already maintain files such as:

```text
CONTEXT.md
CONTEXT-MAP.md
docs/adr/
specs/
plans/
```

Those stay authoritative in Git.

A dev log should reference them:

```markdown
## Repository Knowledge

- New ADR: `docs/adr/0008-separate-transport-state.md`
- Updated domain context: `CONTEXT.md`
```

rather than copying their contents into Obsidian.

This works well with engineering skill sets that already produce repo-local context, ADRs, specs, plans, tests, and review artifacts.

## Repository binding

Each repo-backed project stores a normalized repository identity in frontmatter:

```yaml
---
type: project
project: FilmEnlarger
status: active
priority: P1
repository: github.com/example/FilmEnlarger
updated: 2026-09-14
---
```

The skill resolves the current Git checkout using:

```bash
git rev-parse --show-toplevel
git remote get-url origin
```

and normalizes SSH/HTTPS remotes to one stable identity. This keeps dev logs from one repo attached to the correct Obsidian project.

## Priority model

Projects and tasks use the same four levels:

```text
P0 = Critical
P1 = High
P2 = Normal
P3 = Low
```

Project priority appears in the project index and is mirrored in `Global Todo.md`.

Task priority stays human-editable directly in `Todo.md`:

```markdown
- [ ] [P0] Fix film transport jam
- [ ] [P1] Complete automatic calibration
- [ ] [P2] Add recovery behavior
- [ ] [P3] Explore automatic dust detection
```

Humans and agents may both edit these files. Agents are instructed not to casually override human priority or wording.

## Reports are on demand

The skill does **not** automatically create daily or weekly reports.

Ask when useful:

```text
What's the todo list for FilmEnlarger?
```

```text
What are my highest-priority tasks across active projects?
```

```text
Give me the weekly engineering report for FilmEnlarger. Don't save it.
```

Reports are generated from the underlying todo/dev-log state and stay in chat unless you explicitly ask to save them.

## Logging is deliberate

The skill also does **not** automatically record every coding discussion or invocation of another skill.

After a meaningful work unit, ask:

```text
Use obsidian-project-brain to log this session.
```

or:

```text
Update the project brain with what we accomplished. Update Todo and today's dev log. Reference any repo ADRs/context/specs we created; don't duplicate them into Obsidian.
```

This keeps the vault concise and human-owned.

---

# Setup

## 1. Choose your Obsidian vault

Any local Obsidian vault or Markdown directory works.

You do not need to create the project structure manually; the skill can initialize it.

## 2. Configure the vault path

### Option A — environment variable

Add to `~/.zshrc` on macOS:

```bash
export OBSIDIAN_VAULT="$HOME/Documents/Obsidian/MyVault"
```

Reload:

```bash
source ~/.zshrc
```

### Option B — config file

```bash
mkdir -p ~/.config/agent-obsidian
printf '%s\n' "$HOME/Documents/Obsidian/MyVault" \
  > ~/.config/agent-obsidian/vault-path
```

The skill checks `OBSIDIAN_VAULT` first, then the config file.

## 3. Install the skill

Install from this repository with the `skills` CLI:

```bash
npx skills@latest add crispyfisherman/obsidian-project-brain \
  --global \
  --agent claude-code \
  --agent codex \
  --agent gemini-cli
```

Choose **Symlink** when prompted if you want all agents to use the same installed copy.

Typical global locations are:

```text
Claude Code: ~/.claude/skills/obsidian-project-brain
Codex:       ~/.codex/skills/obsidian-project-brain
Gemini CLI:  ~/.gemini/skills/obsidian-project-brain
```

Verify:

```bash
ls -la ~/.claude/skills/obsidian-project-brain
ls -la ~/.codex/skills/obsidian-project-brain
ls -la ~/.gemini/skills/obsidian-project-brain
```

## 4. Initialize the global dashboard

Ask an installed agent:

```text
Use obsidian-project-brain to initialize Global Todo if it doesn't exist. Preserve my existing vault conventions.
```

The initial dashboard should stay minimal:

```markdown
# Global Todo

## Active Projects

## Paused

## Completed
```

## 5. Initialize a project

From the root of a Git repository:

```text
Use obsidian-project-brain to initialize this repository as an active P1 project.
Bind it to this Git remote, create the project index and Todo, and add it to Global Todo.
Do not invent a backlog. If this repo has CONTEXT.md, ADRs, specs, or plans, link to them from the project index instead of copying them.
```

The skill creates the minimum useful project tracking surface first. `Dev Logs/`, `Notes/`, and `Tasks/` can appear later when needed.

## 6. Normal usage

### Human editing

Open the vault normally in Obsidian and edit:

```text
Global Todo.md
Projects/<Project>/Todo.md
```

These are intentionally plain Markdown and easy to maintain by hand.

### Log a coding session

```text
Use obsidian-project-brain to log this session.
Update Todo and today's dev log based on what actually changed.
Reference any repo ADRs, CONTEXT files, specs, plans, PRs, or commits that matter.
```

### Relearn an old codebase

```text
I haven't worked on this repository for a while.
Use the project brain and the actual repository to help me understand current work, recent history, and the canonical repo docs I should read.
```

### Project todo

```text
What's the todo list for FilmEnlarger?
```

### Cross-project priority

```text
What projects am I currently working on, ordered by priority?
```

```text
What are my highest-priority tasks across active projects?
```

### Weekly report

```text
Give me a weekly engineering report for FilmEnlarger for this week.
Include completed work, in-progress work, repo knowledge/decisions referenced during the week, blockers, remaining work, and recommended next focus.
Don't save the report to the vault.
```

---

# Design notes

The ownership model is intentionally simple:

```text
Current personal work state  → Global Todo.md / Todo.md
Chronological progress       → Dev Logs/
Personal learning            → Notes/
Canonical engineering truth  → Git repository
```

Obsidian is a project cockpit and journal, not a duplicate engineering knowledge base.

This skill uses ordinary Markdown, YAML properties, and Obsidian wikilinks. No Obsidian community plugin, database, vector store, or semantic index is required.

## Agent Skills compatibility

This repository follows the Agent Skills format: a skill directory contains `SKILL.md` with `name` and `description` YAML frontmatter, with optional `references/` resources loaded as needed.

## License

MIT
