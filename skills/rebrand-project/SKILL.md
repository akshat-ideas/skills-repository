---
name: rebrand-project
description: Use when the user says "remove python" or "rename the project".
---

# Rebrand Project

## Overview

Two independent operations, often run together:
1. **Python/FastAPI cleanup** — remove dead env vars and config left over from a removed FastAPI service
2. **Project rename** — replace all hardcoded project name strings across source, config, and HTML

Always run `pnpm type:check` after making changes.

---

## Operation 1: Remove Python/FastAPI Remnants

### Discovery step

Run this first to find every reference before touching anything:

```bash
grep -r --include="*.ts" --include="*.tsx" --include="*.json" --include="*.yml" --include="*.env" \
  -i "fastapi\|python\|FASTAPI" . --exclude-dir=node_modules
```

Remove every hit that isn't inside `node_modules`.

### Files and directories to delete

| Path | Action |
|------|--------|
| `services/` | Delete entire directory (all Python source files) |
| `.python-version` | Delete file |
| `uv.lock` | Delete file |
| `pyproject.toml` | Delete file |

### Files to edit

| File | What to remove |
|------|----------------|
| `.env` | `FASTAPI_PORT=3002` line and `FASTAPI_SERVICE_URL=http://localhost:${FASTAPI_PORT}` line |
| `src/server/env.ts` | Any `FASTAPI_PORT` or `FASTAPI_SERVICE_URL` fields in the Zod `EnvSchema` object |
| `docker-compose.yml` | Any `fastapi` service block |

---

## Operation 2: Rename the Project

### Step 1 — Ask for the new name

Before touching any file, ask the user:
- **New slug** (kebab-case, used in `package.json` name, URLs, HTML meta) — e.g. `intel-briefer`
- **New display name** (Title Case, used in UI strings and copy) — e.g. `Intel Briefer`
- **New allowed email domains** (replaces the nodwin/nodwingaming entries), or confirm to keep existing

### Step 2 — Discovery

Run a grep to find every occurrence before changing anything:

```bash
grep -rn --include="*.ts" --include="*.tsx" --include="*.json" --include="*.html" --include="*.md" \
  --include="*.yml" --include="*.env" \
  -i "meeting-prep-tool\|meeting prep tool\|nodwin\|nodwingaming" . --exclude-dir=node_modules
```

### Step 3 — Files to update

| File | What to change |
|------|---------------|
| `package.json` | `"name"` field → new slug |
| `src/client/index.html` | `<title>`, `<meta name="description">`, `<meta name="keywords">` |
| `src/client/components/app-navbar.tsx` | Hardcoded name string in the navbar logo area |
| `src/client/pages/login-page.tsx` | Login/signup page description strings |
| `src/core/auth/allowed-signup-domains.ts` | Domain list (nodwin.com, nodwingaming.com, etc.) |
| `src/core/auth/allowed-signup-domains.test.ts` | Test assertions referencing nodwin domains |
| `README.md` | Project name references |
| `AGENTS.md` | Any project name references |
| `docker-compose.yml` | Service labels or comments referencing old name |

### Step 4 — Make changes

Use exact string replacement (not regex) for each file. For the slug (kebab-case), replace `meeting-prep-tool`. For the display name, replace `Meeting Prep Tool`. For domains, replace the `allowedDomains` array contents.

### Step 5 — Verify

```bash
# Check for missed occurrences
grep -rn --include="*.ts" --include="*.tsx" --include="*.json" --include="*.html" --include="*.md" \
  -i "meeting-prep-tool\|meeting prep tool\|nodwin\|nodwingaming" . --exclude-dir=node_modules

# Type-check passes
pnpm type:check
```

Zero grep hits and a passing type-check = done.

---

## Common Mistakes

- **Forgetting `index.html`** — it has three meta tags (`description`, `keywords`, `title`) all using the old name.
- **Forgetting the test file** — `allowed-signup-domains.test.ts` has hardcoded domain assertions that will fail if you only update the source file.
- **Only updating the slug but not the display name** — `login-page.tsx` uses the display name in sentence copy ("Sign in to your X account").
- **Not asking about domains first** — the allowed-domains list is business logic, not just a string rename; confirm with the user before changing it.