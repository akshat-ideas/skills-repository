---
name: writing-readmes
description: Use when creating or rewriting a README.md for an application project - reads the codebase first, asks only what's missing, produces the 7 standard sections (What it does, Install, Use, Configure, Backup, Restore, Uninstall) in plain markdown.
---

# Writing READMEs

## When to use

- User asks to write or rewrite a README.md for an application project
- Working directory has an application manifest (package.json, pyproject.toml, Cargo.toml, go.mod, *.csproj, pubspec.yaml, composer.json, Gemfile, pom.xml)

## When NOT to use

- **Libraries or frameworks** — different lifecycle, different conventions
- **No code exists yet** — nothing to discover
- **Single-section edits** — full workflow is overkill
- **User explicitly wants non-standard structure** — defer to user

## Workflow

**Step 1 — Discover.** Read in this order, stop early when enough: manifest files → existing README (for tone only, not structure) → entry points → config files → persistence markers (decides if backup/restore apply) → install/build scripts.

**Step 2 — Clarify.** Ask via the `question` tool **only** for gaps code can't answer: multiple deployment targets, ambiguous audience, unclear backup location, update ≠ install. Cap 3 questions. Never ask things in package.json. Always use `question`, never prose.

**Step 3 — Write.** Produce `./README.md` (overwriting if needed). Run the Quality Checklist mentally first; fix failures before producing output.

**Step 4 — Suggest.** Propose 1-3 follow-up improvements via `question` (e.g., "Add a Quick Start under How to use?", "Run the install commands to verify they work?", "Add a Troubleshooting section after Uninstall?").

## The 7-Section Template

Always in this order:

| # | Section | Audience focus | When N/A |
|---|---------|----------------|----------|
| 1 | **What this does** — 1-3 sentences: purpose + primary use case | End-user first | Never |
| 2 | **How to install** — prereqs, install commands, verification step | End-user (binary) + Developer (source) | Never |
| 3 | **How to use** — quick start + most common operation | End-user | Never |
| 4 | **How to configure / update** — config location, common settings, update command | End-user + Operator | If no config AND no updates, mark N/A with reason |
| 5 | **How to backup** — what, where, command | Operator | If stateless, mark N/A with reason |
| 6 | **How to restore** — restore command + verification | Operator | Same rule as backup |
| 7 | **How to uninstall** — removal + cleanup | End-user + Operator | Never |

Header format: `## How to <verb>`. N/A format:

```markdown
## How to backup

Not applicable — this app is stateless and stores no user data.
```

## Quality Checklist (run before writing)

- [ ] Project name appears in title or first sentence
- [ ] Every command verified against actual manifest/script (not invented)
- [ ] No placeholder text (`TODO`, `TBD`, `lorem`, `...`)
- [ ] Every section has at least one concrete piece of information
- [ ] N/A sections explain *why* in one sentence
- [ ] No badges, screenshots, diagrams, or images
- [ ] Both audiences served (end-user first, operator for backup/restore, developer source-build in Install)
- [ ] Plain markdown only — no HTML, no fancy formatting

## Red Flags — STOP and fix

- Installing without reading the manifest first
- Asking questions answerable from the manifest
- Inventing commands not present in the project
- Skipping the Quality Checklist
- Producing fewer than 7 sections
- Marking a section "N/A" without explaining why
- Omitting a section entirely instead of marking N/A (vacuously passes the no-fabrication check but loses information)
- Adding badges, screenshots, or diagrams
- Using prose questions instead of the `question` tool
- Preserving existing README's structure instead of rewriting into the standard 7
- Writing the same install command for end-user and developer without acknowledging they differ

**All of these mean: delete the README, fix the issue, restart the workflow.**
