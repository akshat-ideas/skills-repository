# Skills repository

A collection of skills for AI coding agents. Each skill is a self-contained markdown reference (with optional helper scripts) that an agent can load on demand to apply proven techniques, patterns, or workflows.

Skills cover engineering process (TDD, debugging, planning), code quality, design, document generation, and platform-specific patterns. New skills are added on an ongoing basis; see **How to contribute** below to write your own.

## How to install

### Prerequisites

- Git
- One or more supported agents:
  - **Claude Code**
  - **Codex / opencode** (uses the Codex skill layout at `~/.agents/skills/`)
  - **Any other agent** that reads skills from a local directory

The repository itself is pure markdown plus a handful of helper scripts (mostly Python under `skills/*/scripts/`). No runtime is installed.

### Clone the repository

Go to your root of your claude projects/ home / where you want your skills to be placed. Open a terminal here

```bash
git clone https://github.com/akshat-ideas/skills-repository skills-repository
cd skills-repository
```

### Install skills for your agent

The repository ships skills under `skills/<name>/SKILL.md`. Copy or symlink that directory into the location your agent scans for skills.

**Claude Code** (default install path):

```bash
# Linux / macOS / Git Bash
mkdir -p ~/.claude/skills
cp -r skills/* ~/.claude/skills/

# Windows PowerShell
New-Item -ItemType Directory -Force -Path $HOME\.claude\skills
Copy-Item -Recurse -Force skills\* $HOME\.claude\skills\
```

**Codex / opencode** (`~/.agents/skills/`):

```bash
# Linux / macOS / Git Bash
mkdir -p ~/.agents/skills
cp -r skills/* ~/.agents/skills/

# Windows PowerShell
New-Item -ItemType Directory -Force -Path $HOME\.agents\skills
Copy-Item -Recurse -Force skills\* $HOME\.agents\skills\
```

**Prefer a symlink** so updates to the clone flow through automatically:

```bash
# Linux / macOS / Git Bash
ln -s "$(pwd)/skills" ~/.claude/skills-novalabs

# Windows PowerShell (requires admin or Developer Mode)
New-Item -ItemType SymbolicLink -Path $HOME\.claude\skills-novalabs -TargetType Directory -Target "$(Resolve-Path skills)"
```

After copying, the agent can discover every skill by name. The `using-superpowers` skill governs discovery and should be the first one invoked.

### Verify

Open any skill file and confirm the frontmatter parses:

```bash
head -5 skills/using-superpowers/SKILL.md
```

Expected output starts with `---` followed by `name: using-superpowers`. If your agent is wired up, asking it to "use the brainstorming skill" or "use the test-driven-development skill" will load the file on demand.

## How to use

Each skill is a `SKILL.md` file with a YAML frontmatter block containing a `name` and a `description`. Agents that follow the [agentskills.io spec](https://agentskills.io/specification) match the description against the current task and load the matching skill.

### Quick start

1. Install the skills for your agent (see **How to install**).
2. Start a conversation with your agent.
3. When a task matches a skill's trigger conditions, the agent loads the skill before responding.

### Skill catalog

#### Process and meta

- `using-superpowers` — Skill discovery discipline; load before any other response.
- `brainstorming` — Turn ideas into designs before implementation.
- `writing-skills` — Author new skills using TDD applied to documentation.
- `verification-before-completion` — Evidence before success claims.
- `grill-me` — Stress-test a plan or design.
- `handoff` — Compact a conversation for another agent.
- `teach` — Teach the user a new concept across sessions.

#### Development workflow

- `test-driven-development` — RED-GREEN-REFACTOR.
- `systematic-debugging` — Root-cause-first diagnosis.
- `diagnosing-bugs` — Loop for hard bugs and regressions.
- `writing-plans` — Plan a multi-step task before touching code.
- `executing-plans` — Execute a plan in a separate session with checkpoints.
- `subagent-driven-development` — Execute a plan with independent tasks.
- `dispatching-parallel-agents` — 2+ independent tasks, no shared state.
- `finishing-a-development-branch` — Merge, PR, or cleanup.
- `receiving-code-review` — Verify review feedback before acting.
- `requesting-code-review` — Pre-merge verification.
- `using-git-worktrees` — Isolate feature work.
- `thermo-nuclear-code-quality-review` — Strict maintainability audit.
- `improve-codebase-architecture` — Find deepening opportunities.
- `deslop` — Strip AI-generated slop from a branch.
- `domain-modeling` — Pin down ubiquitous language.
- `rebrand-project` — Rename or migrate a project.

#### Code quality and patterns

- `vercel-react-best-practices` — React and Next.js performance.
- `vercel-react-native-skills` — React Native and Expo performance.
- `no-use-effect` — Replace `useEffect` with the five idiomatic patterns.
- `web-design-guidelines` — Web UI compliance review.
- `impeccable` — Production-grade frontend interfaces.
- `humanizer` — Strip AI-writing tells from prose.
- `shadcn` — Manage shadcn/ui components and registries.
- `git-guardrails-claude-code` — Hooks to block dangerous git operations.

#### Documentation and specs

- `to-prd` — Conversation to PRD.
- `to-issues` — Plan or spec to tracker issues.
- `writing-readmes` — Author application READMEs.
- `writing-plans` — Implementation plan from a spec.

#### Document file formats

- `docx` — Create, read, edit, and manipulate Word documents.
- `pdf` — Read, extract, merge, split, watermark, and create PDFs.
- `pptx` — Build and edit slide decks.
- `xlsx` — Open, read, edit, and create spreadsheets.

### Loading a skill manually

If your agent does not auto-load skills, point it at a file directly:

```
Read skills/test-driven-development/SKILL.md and follow it for this task.
```

### Contributing a new skill

Use the `writing-skills` skill. It applies test-driven development to process documentation: write a failing pressure scenario, run the baseline, write the minimal skill, then close loopholes. All skills in this repository follow that workflow.

## How to configure

Not applicable — this repository has no configuration files, environment variables, or per-skill settings. Skill behavior is determined entirely by each `SKILL.md`'s frontmatter and content.

### Update

Pull the latest skills:

```bash
git pull
```

If you installed via `cp`, re-copy:

```bash
cp -r skills/* ~/.claude/skills/   # or ~/.agents/skills/ for Codex / opencode
```

If you installed via symlink, the pull is enough.

## How to backup

Not applicable — this repository is content-only. The authoritative copy is the git clone; restoring from a backup means re-cloning. Skills hold no runtime state, user data, or generated artifacts.

## How to restore

Not applicable — see **How to backup**. To restore the skills on a new machine, follow **How to install** from scratch.

## How to uninstall

Remove the skills from the agent's skill directory.

**Claude Code:**

```bash
# Linux / macOS / Git Bash — remove only the skills this repo installed
rm -rf ~/.claude/skills/SKILL-NAME

# Windows PowerShell
Remove-Item -Recurse -Force $HOME\.claude\skills\SKILL-NAME
```

**Codex / opencode:**

```bash
rm -rf ~/.agents/skills/SKILL-NAME
```

If you used a symlink, remove the symlink instead:

```bash
rm ~/.claude/skills-novalabs
```

Optionally remove the cloned repository itself. Substitute the actual clone path on your machine:

```bash
rm -rf /path/to/skills-repository
```

## Troubleshooting

### The agent cannot find a skill

Most agents scan a fixed directory per platform. Confirm the skill landed in the path your agent reads:

- Claude Code: `~/.claude/skills/` (or the symlink you created)
- Codex / opencode: `~/.agents/skills/`

Some agents also read project-local skills (e.g. `.claude/skills/` inside a repo). Those override the user-level directory for that project only.

### Windows symlink fails

`New-Item -ItemType SymbolicLink` requires either an elevated PowerShell or Developer Mode enabled in Windows Settings. If neither is available, fall back to the `Copy-Item` commands in **How to install**, and re-copy after each `git pull`.

### Updates are not picked up

If you installed with `cp`, the agent keeps using the old copy until you copy again:

```bash
cp -r skills/* ~/.claude/skills/   # or ~/.agents/skills/ for Codex / opencode
```

If you installed with a symlink, restart the agent session so it re-reads the directory.

### Two copies of the same skill conflict

When multiple paths contribute skills with the same `name`, the agent's behavior is implementation-defined. List duplicates and remove the older one:

```bash
# Linux / macOS / Git Bash
find ~/.claude/skills ~/.agents/skills -name SKILL.md -path '*/<skill-name>/*' 2>/dev/null

# Windows PowerShell
Get-ChildItem -Recurse -Filter SKILL.md -Path $HOME\.claude\skills,$HOME\.agents\skills
```

### Frontmatter parse error

Each `SKILL.md` must start with a YAML block delimited by `---`, containing `name` (letters, numbers, hyphens only) and `description` (max 1024 characters). Validate by reading the first five lines:

```bash
head -5 skills/SKILL-NAME/SKILL.md
```

If the agent refuses to load the skill, check that no character has slipped past 1024 in `description`, and that `name` contains no parentheses or underscores.

## License

Individual skills retain their original licenses — see each `skills/<name>/LICENSE.txt` where present. Most are © Anthropic, PBC and governed by your agreement with Anthropic regarding use of Anthropic's services.
