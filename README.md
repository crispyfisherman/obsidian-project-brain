# obsidian-project-brain

A portable Agent Skill for maintaining an Obsidian vault as a **human + coding-agent shared development workspace**.

It keeps the parts of software work that are useful beyond the current chat/session:

- active projects and project priority;
- project todos, task priority, blockers, and progress;
- current architecture;
- codebase maps for fast relearning;
- technical decisions and their reasoning;
- concise development logs;
- on-demand daily/weekly/project reports.

It is deliberately **not** a coding methodology and does not replace planning, TDD, debugging, review, Git history, or the repository itself.

## Knowledge model

```text
Obsidian Vault/
├── Global Todo.md
└── Projects/
    └── <Project>/
        ├── <Project> Index.md
        ├── Todo.md
        ├── Architecture.md
        ├── Codebase Map.md
        ├── Decisions/
        ├── Dev Logs/
        └── Tasks/
```

### Why this structure?

- `Global Todo.md` answers **what projects am I working on?**
- `<Project> Index.md` is the project entry point and owns project status/priority.
- `Todo.md` is the canonical human-friendly task state for that project.
- `Architecture.md` explains how the system works **now**.
- `Codebase Map.md` makes old or unfamiliar codebases fast to navigate again.
- `Decisions/` explains **why** important technical choices were made.
- `Dev Logs/` records meaningful chronological development history.
- `Tasks/` is only for tasks complex enough to need their own note.

Humans and agents can edit the same files.

## Priority model

Both projects and tasks use:

```text
P0 = Critical
P1 = High
P2 = Normal
P3 = Low
```

Project priority is stored in the project index frontmatter and mirrored in `Global Todo.md`.

Task priority is normally stored inline in `Todo.md`:

```markdown
- [ ] [P0] Fix film transport jam
- [ ] [P1] Complete automatic calibration
- [ ] [P2] Add recovery behavior
- [ ] [P3] Explore automatic dust detection
```

## Reports are on demand

The skill does **not** create automatic daily or weekly reports.

Ask for them when useful:

```text
What's the todo list for FilmEnlarger?
```

```text
What are my highest-priority tasks across active projects?
```

```text
Give me the weekly engineering report for FilmEnlarger.
```

The report is generated from the underlying todo/dev-log/decision data and stays in chat unless you explicitly ask to save it.

---

# Setup

## 1. Create or choose an Obsidian vault

Any normal local Obsidian vault works.

You do not need to create the project structure manually; the skill can initialize it when you ask.

## 2. Configure the vault path

### Option A — environment variable

Add this to `~/.zshrc` on macOS:

```bash
export OBSIDIAN_VAULT="$HOME/Documents/Obsidian/MyVault"
```

Then reload your shell:

```bash
source ~/.zshrc
```

### Option B — small config file

```bash
mkdir -p ~/.config/agent-obsidian
printf '%s\n' "$HOME/Documents/Obsidian/MyVault" \
  > ~/.config/agent-obsidian/vault-path
```

Replace the example path with your real vault path.

The skill checks `OBSIDIAN_VAULT` first, then the config file.

## 3. Install this skill globally

Once this repository is on GitHub, install the same skill into Claude Code, Codex, and Gemini CLI using the open `skills` CLI:

```bash
npx skills@latest add crispyfisherman/obsidian-project-brain \
  --global \
  --agent claude-code \
  --agent codex \
  --agent gemini-cli
```

Choose **Symlink** when prompted. That keeps one canonical installed copy rather than independent copies per agent.

The `skills` CLI currently recognizes these global locations:

```text
Claude Code: ~/.claude/skills/obsidian-project-brain
Codex:       ~/.codex/skills/obsidian-project-brain
Gemini CLI:  ~/.gemini/skills/obsidian-project-brain
```

Verify after installation:

```bash
ls -la ~/.claude/skills/obsidian-project-brain
ls -la ~/.codex/skills/obsidian-project-brain
ls -la ~/.gemini/skills/obsidian-project-brain
```

If an installer regression leaves the canonical skill installed but misses one agent-specific symlink, create only the missing symlink manually after confirming the canonical install path.

## 4. Initialize your vault's global dashboard

Ask any installed agent:

```text
Use the obsidian-project-brain skill to initialize my vault's Global Todo if it doesn't exist yet. Preserve my existing vault conventions.
```

The initial file should stay minimal:

```markdown
# Global Todo

## Active Projects

## Paused

## Completed
```

## 5. Initialize a project

From the root of a repository:

```text
Use the obsidian-project-brain skill to initialize this project in my vault.
Inspect the repository first and preserve existing vault conventions.
Create only the minimum project index, Todo, Architecture, and Codebase Map.
Add the project to Global Todo with priority P2 unless I already specified a priority.
Do not invent a task backlog.
```

If you want a different initial priority, say so explicitly, for example:

```text
Initialize this project in the vault as an active P1 project.
```

## 6. Normal usage

### Human editing

Open the vault normally in Obsidian and edit:

```text
Global Todo.md
Projects/<Project>/Todo.md
```

Those files are intentionally plain Markdown and easy to maintain by hand.

### Agent updates after coding

```text
Update the vault with the durable changes from this session.
Update the dev log and todo state where needed.
Only update Architecture, Codebase Map, or Decisions if something meaningful actually changed.
```

### Relearn an old codebase

```text
I haven't worked on this repository for a while.
Use the vault and the actual repository to help me relearn the architecture, important code paths, recent decisions, and current todo list.
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
Include completed work, in-progress work, technical decisions, architecture changes, blockers, remaining work, and recommended next focus.
Don't save the report to the vault.
```

---

# Publish this as a GitHub repository

The recommended repository name is **`obsidian-project-brain`** because the Agent Skills specification requires the skill's `name` to match its directory name.

The repository can contain the skill directly at the root:

```text
obsidian-project-brain/
├── SKILL.md
├── README.md
├── LICENSE
└── references/
    └── note-templates.md
```

## Create the local Git repository

From this folder:

```bash
git init
git branch -M main
git add .
git commit -m "Initial obsidian-project-brain agent skill"
```

### With GitHub CLI

If `gh` is installed and authenticated:

```bash
gh repo create obsidian-project-brain \
  --public \
  --source=. \
  --remote=origin \
  --push
```

Use `--private` instead if you do not want the skill public yet.

### Without GitHub CLI

Create an empty `obsidian-project-brain` repository on GitHub, then:

```bash
git remote add origin git@github.com:YOUR_GITHUB_USERNAME/obsidian-project-brain.git
git push -u origin main
```

After it is pushed, test discovery before installing:

```bash
npx skills@latest add YOUR_GITHUB_USERNAME/obsidian-project-brain --list
```

Then install it globally using the command in the Setup section.

---

# Skill design notes

The design intentionally keeps current state and history separate:

```text
Current truth      → Architecture.md / Codebase Map.md / Todo.md
Historical reason  → Decisions/
Chronological work → Dev Logs/
```

The repository always remains authoritative for what the code actually does.

This skill uses ordinary Markdown, YAML properties, and Obsidian wikilinks. No Obsidian community plugin, database, vector store, or semantic index is required.

## Agent Skills compatibility

This repository follows the Agent Skills format: a skill directory contains `SKILL.md` with `name` and `description` YAML frontmatter, with optional `references/` resources loaded as needed.

## License

MIT
