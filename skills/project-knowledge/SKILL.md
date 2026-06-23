---
name: project-knowledge
description: >
  Maintains a living PROJECT_KNOWLEDGE.md file for any coding project.
  Use this skill whenever the user asks to "update project docs", "log what we did",
  "sync project knowledge", "update the project file", "what have we done so far",
  "start a new session", or any time the user wants to record progress, changes,
  fixes, removals, or tech decisions made during a coding session.
  Also trigger at the END of any coding session if the user says "we're done" or "that's it for now"
  — proactively offer to update the project knowledge file.
  This skill is critical for continuity: it ensures any new AI session can instantly
  understand the project state without re-asking or duplicating work.
---

# Project Knowledge Skill

This skill manages a `PROJECT_KNOWLEDGE.md` file that lives at the root of the user's project.
It is the single source of truth about the project — what exists, what changed, what was removed, and why.

---

## When This Skill is Invoked

There are three modes:

| Mode | When | Action |
|------|------|--------|
| **CREATE** | No `PROJECT_KNOWLEDGE.md` exists yet | Interview the user, then generate the full file |
| **UPDATE** | File exists + user reports new changes | Read current file, apply changes, write back |
| **READ** | New session starts / user says "catch me up" | Read and summarize the file to orient the AI |

---

## Step 1 — Detect Mode

Check if `PROJECT_KNOWLEDGE.md` exists in the current directory or project root:

```bash
find . -maxdepth 2 -name "PROJECT_KNOWLEDGE.md" 2>/dev/null
```

- **Found** → Load it, enter UPDATE or READ mode
- **Not found** → Enter CREATE mode

---

## Step 2A — CREATE Mode (First Time)

### Phase 1: Auto-Analyze the Project

Before asking the user anything, scan the project yourself. Run these in order:

**1. Get the full file/folder structure:**
```bash
find . -not -path '*/node_modules/*' -not -path '*/.git/*' -not -path '*/__pycache__/*' \
  -not -path '*/dist/*' -not -path '*/.next/*' -not -path '*/build/*' \
  -not -path '*/venv/*' -not -path '*/.venv/*' | sort | head -120
```

**2. Detect tech stack from config files — read whichever exist:**
- `package.json` → name, version, dependencies, devDependencies, scripts
- `requirements.txt` / `pyproject.toml` / `Pipfile` → Python dependencies
- `go.mod` → Go modules
- `Cargo.toml` → Rust
- `pom.xml` / `build.gradle` → Java/Kotlin
- `composer.json` → PHP
- `Gemfile` → Ruby
- `*.csproj` / `*.sln` → C#/.NET
- `docker-compose.yml` / `Dockerfile` → containerization
- `.env.example` → environment variables and services used

**3. Detect framework and purpose from key files:**
- `next.config.*`, `vite.config.*`, `astro.config.*` → frontend framework
- `main.py`, `app.py`, `manage.py`, `server.ts`, `index.ts` → entry points
- Read entry point files (first 60 lines each) to understand what the app does
- Check `README.md` if it exists

**4. Scan for features, systems, and workflows:**

*Features & Screens:*
- List top-level folders under `src/`, `app/`, `pages/`, `routes/`, `api/`, `components/`, `modules/`, `views/`, `screens/`
- Read route/controller/page files briefly (first 50 lines each) to infer what each endpoint or screen does
- Map each route/page to a user-facing feature (e.g. `/api/auth/login` → Authentication, `pages/dashboard.tsx` → Dashboard)

*Systems (cross-cutting infrastructure) — grep for these patterns:*
```bash
# Auth
grep -rl "auth\|login\|logout\|session\|jwt\|token\|passport\|oauth" \
  --include="*.ts" --include="*.js" --include="*.py" \
  --exclude-dir=node_modules --exclude-dir=.git . -l 2>/dev/null | head -8

# Database / ORM
grep -rl "prisma\|mongoose\|sequelize\|typeorm\|drizzle\|sqlalchemy\|knex" \
  --include="*.ts" --include="*.js" --include="*.py" \
  --exclude-dir=node_modules . -l 2>/dev/null | head -5

# Email / notifications
grep -rl "nodemailer\|sendgrid\|resend\|mailgun\|twilio\|firebase" \
  --include="*.ts" --include="*.js" --include="*.py" \
  --exclude-dir=node_modules . -l 2>/dev/null | head -5

# Payments
grep -rl "stripe\|paypal\|paddle\|lemonsqueezy\|checkout" \
  --include="*.ts" --include="*.js" --include="*.py" \
  --exclude-dir=node_modules . -l 2>/dev/null | head -5

# File upload / storage
grep -rl "multer\|cloudinary\|s3\|uploadthing\|supabase.*storage\|minio" \
  --include="*.ts" --include="*.js" --include="*.py" \
  --exclude-dir=node_modules . -l 2>/dev/null | head -5

# Background jobs / queues
grep -rl "bullmq\|bull\|celery\|cron\|queue\|worker" \
  --include="*.ts" --include="*.js" --include="*.py" \
  --exclude-dir=node_modules . -l 2>/dev/null | head -5

# Realtime / websockets
grep -rl "socket\.io\|websocket\|pusher\|ably\|supabase.*realtime" \
  --include="*.ts" --include="*.js" --include="*.py" \
  --exclude-dir=node_modules . -l 2>/dev/null | head -5

# AI / LLM integrations
grep -rl "openai\|anthropic\|langchain\|ollama\|gemini\|huggingface" \
  --include="*.ts" --include="*.js" --include="*.py" \
  --exclude-dir=node_modules . -l 2>/dev/null | head -5
```

For each detected system, read the relevant file briefly to understand how it's wired up (e.g. which auth strategy, what DB schema looks like, which storage provider).

*Workflows (multi-step flows) — reconstruct by tracing related files:*
- Read middleware stack and router files to find request lifecycle
- Look for multi-step patterns: signup → verify → onboarding, checkout → webhook → access
- Check for files named `flow`, `pipeline`, `process`, `service`, `wizard`, `step`
- Frontend: look for multi-step forms, wizard components, or sequential page routes
- Backend: look for chained middleware, event listeners, or job sequences

For each workflow found, describe it as a linear sequence of steps, e.g.:
> User Registration: `POST /auth/register` → send verification email → `GET /auth/verify/:token` → create user profile → redirect to onboarding

**5. Inventory server actions / API routes / controllers:**

Scan for every server action, API route, or controller function — these are the backbone of the app:

```bash
# Next.js server actions
grep -rl ""use server"\|'use server'" --include="*.ts" --include="*.tsx"   --exclude-dir=node_modules . -l 2>/dev/null

# Next.js API routes
find . -path '*/api/*/route.ts' -o -path '*/api/*/route.js' 2>/dev/null | grep -v node_modules

# Express/Fastify/Hono routes
grep -rn "router\.\(get\|post\|put\|patch\|delete\)\|app\.\(get\|post\|put\)"   --include="*.ts" --include="*.js" --exclude-dir=node_modules . 2>/dev/null | head -30

# Python (FastAPI / Django / Flask)
grep -rn "@app\.\|@router\.\|@api_view\|path("   --include="*.py" . 2>/dev/null | head -30
```

For each action/route found, read its signature and first 20 lines to understand:
- What it accepts (params, body shape)
- What it returns
- Whether it's auth-gated

Document each as: `actionName(params)` — what it does, auth required Y/N

**6. Inventory hooks, composables, or service functions:**

```bash
# React hooks
find . -path '*/hooks/*' -name '*.ts' -o -path '*/hooks/*' -name '*.tsx' 2>/dev/null   | grep -v node_modules

# Custom hook definitions in any file
grep -rn "^export.*function use[A-Z]\|^export const use[A-Z]"   --include="*.ts" --include="*.tsx" --exclude-dir=node_modules . 2>/dev/null | head -30

# Python services / utils
find . -path '*/services/*' -o -path '*/utils/*' 2>/dev/null | grep '\.py$' | grep -v node_modules
```

For each hook/service, note: what data it fetches or manages, what side effects it has.

**7. Inventory components:**

```bash
# List all component files
find . \( -path '*/components/*' -o -path '*/ui/*' \)   \( -name '*.tsx' -o -name '*.jsx' \)   -not -path '*/node_modules/*' 2>/dev/null | sort
```

Group by folder. Note purpose of each non-trivial component in one line.

**8. Extract environment variables:**

```bash
# From .env.example or .env.local.example
cat .env.example 2>/dev/null || cat .env.local.example 2>/dev/null || cat .env.sample 2>/dev/null

# Grep for process.env / env() usage across the codebase
grep -rn "process\.env\.\|env(\." --include="*.ts" --include="*.js" --include="*.tsx"   --exclude-dir=node_modules . 2>/dev/null | grep -oP 'process\.env\.\K\w+|env\(.\K[^"]+(?=")'   | sort -u | head -40
```

List every variable with: name, which file uses it, what service/feature it enables.

**9. Extract dev / build commands:**

```bash
# From package.json scripts
cat package.json 2>/dev/null | python3 -c "import sys,json; d=json.load(sys.stdin); [print(f'{k}: {v}') for k,v in d.get('scripts',{}).items()]"

# From Makefile
cat Makefile 2>/dev/null | grep '^[a-zA-Z]' | head -20

# From pyproject.toml
grep -A2 '\[tool.taskipy\]\|\[tool.poe\]' pyproject.toml 2>/dev/null
```

**10. Detect any existing docs:**
```bash
find . -maxdepth 2 -name "*.md" -not -path '*/node_modules/*'
```

---

### Phase 2: Build a Findings Summary

After scanning, internally compile what you know vs. what's still unclear:

**Can usually auto-detect:**
- Language(s), runtime, framework(s)
- All installed libraries (from lock/config files)
- Folder structure and entry points
- Features / screens (from routes, pages, components)
- Systems in place (auth, DB, payments, email, queues, realtime, AI, storage)
- Workflows (from middleware chains, route sequences, multi-step components)
- Every server action / API route / controller — name, params, auth requirement
- Every hook / composable / service function — name and what it manages
- Full component inventory grouped by folder
- Environment variables (from .env.example + grep)
- Dev/build commands (from package.json scripts or Makefile)

**Often needs user input:**
- Project purpose (if no README or it's vague)
- What's *actually working* vs. scaffolded/incomplete
- What was recently removed or deprecated
- Known bugs
- Current goal / next milestone

---

### Phase 3: Ask Only What You Couldn't Find

Present your findings first, then ask only about the gaps. Format it like this:

> **Here's what I found about your project:**
> - **Name:** [detected or "couldn't detect"]
> - **Stack:** [languages, frameworks, key libraries]
> - **Systems detected:** [e.g. Auth (JWT), DB (Prisma/Postgres), Email (Resend), Payments (Stripe)]
> - **Features:** [list of screens/routes/modules detected]
> - **Workflows:** [e.g. "User registration → email verify → onboarding", "POST /api/upload → S3 → DB record"]
> - **Server actions / routes:** [count + names detected]
> - **Hooks / services:** [count + names detected]
> - **Components:** [count + folder breakdown]
> - **Env vars found:** [list of variable names]
> - **Dev commands:** [list from scripts]
>
> **A few things I couldn't determine on my own:**
> 1. [e.g. "What does this app do? No README found."]
> 2. [e.g. "Are all these routes working or are some still WIP?"]
> 3. [e.g. "What's the data shape that your external system writes to the DB?"]
> 4. [e.g. "Any known bugs right now?"]
> 5. [e.g. "What's the current goal or next milestone?"]

Keep questions to the absolute minimum. If the code makes it clear, don't ask.

Then generate the full `PROJECT_KNOWLEDGE.md` using the template in the **Template** section below.

---

## Step 2B — UPDATE Mode (Ongoing Sessions)

### Phase 1: Auto-Detect What Changed

Before asking anything, try to detect changes since the last update:

```bash
# Check git log for commits since last documented date
git log --oneline --since="[last session date from PROJECT_KNOWLEDGE.md]" 2>/dev/null

# Check recently modified files (last 24h or since last session)
find . -newer PROJECT_KNOWLEDGE.md -not -path '*/node_modules/*' \
  -not -path '*/.git/*' -not -path '*/dist/*' -not -path '*/__pycache__/*' \
  -type f 2>/dev/null | head -40

# If git available, show changed files
git diff --name-only HEAD~5..HEAD 2>/dev/null || git status --short 2>/dev/null
```

Read any newly modified source files briefly (top 40 lines) to understand what changed.

Check if `package.json` or dependency files changed → detect new installs.

### Phase 2: Present Findings + Ask for Gaps

> **Detected since last session:**
> - Modified files: [list]
> - New dependencies: [if any]
> - Likely changes: [inferred from file names / git messages]
>
> **Can you confirm or add anything I missed?** (fixes, removals, decisions not obvious from code)

Then apply all changes to the file.

Accept free-form input. Extract and sort into the right sections:
- New features → add to `## Features`
- Removed code/features → add to `## Removed`
- Bug fixes → add to `## Fixed`
- New packages/tools → add to `## Tech Stack`
- Structural changes → update `## Project Structure`
- New issues → add to `## Known Issues`
- New server actions → add to `## Server Actions / API Routes`
- New hooks or services → add to `## Hooks & Services`
- New components → add to `## Component Inventory`
- New env vars → add to `## Environment Variables`
- Schema changes → update `## Database Schema`
- New workflows → add to `## Workflows`
- External system changes → update `## External Integrations & Data Contracts`

Always append a new entry to `## Session Log` with today's date and a 1–3 line summary of what changed.

Never delete history. Only add to or update existing entries. Mark deprecated items with ~~strikethrough~~ rather than removing them.

---

## Step 2C — READ Mode (New Session Orientation)

When a new session starts and the user shares or references the `PROJECT_KNOWLEDGE.md`, read it and output:

> A short briefing (under 150 words) covering:
> - What the project is
> - Current tech stack
> - What's been built so far
> - Last session summary
> - What the current goal/next step is

Then say: "Ready to continue — what are we working on?"

---

## Template

Use this exact structure when creating or rewriting `PROJECT_KNOWLEDGE.md`:

```markdown
# PROJECT KNOWLEDGE — [Project Name]

> Last updated: [Date]
> Status: [Active / On Hold / Complete]

---

## What This Project Does
[1–3 sentence description of the project's purpose and target user]

---

## Tech Stack
| Category        | Details                              |
|-----------------|--------------------------------------|
| Language        | e.g. TypeScript, Python              |
| Runtime         | e.g. Node.js 20, Python 3.11         |
| Framework       | e.g. Next.js 14, FastAPI             |
| Database        | e.g. PostgreSQL, SQLite, none        |
| ORM / Query     | e.g. Prisma, SQLAlchemy              |
| Auth            | e.g. NextAuth, JWT, none             |
| Styling         | e.g. Tailwind CSS, SCSS              |
| State Mgmt      | e.g. Zustand, Redux, none            |
| Testing         | e.g. Vitest, Pytest                  |
| Key Libraries   | e.g. axios, zod, pydantic, etc.      |
| Dev Tools       | e.g. ESLint, Prettier, Docker        |
| Deployment      | e.g. Vercel, Railway, local only     |

---

## Project Structure
```
[folder tree — auto-generated from scan]
```
Key files:
- `path/to/file` — what it does

---

## Database Schema
> Tables, key columns, and relations. Source of truth is the ORM schema / migration files — this is a navigable summary.

| Table           | Key Columns                                      | Relations                        |
|-----------------|--------------------------------------------------|----------------------------------|
| [table_name]    | id, col1, col2, col3 (FK → other_table.id), ...  | has_many [x], belongs_to [y]     |

RLS / access rules (if any):
- `[table]` — [who can read/write, e.g. "tenant-scoped: staff can read own business rows only"]

---

## Server Actions / API Routes
> Every server action, API endpoint, or controller. Don't create duplicates — check here first.

| Action / Route                  | Auth Required | What It Does                                      |
|---------------------------------|---------------|---------------------------------------------------|
| `actionName(params)`            | SUPER_ADMIN   | [one-line description]                            |
| `POST /api/route`               | Session       | [one-line description]                            |

---

## Hooks & Services
> Every custom hook, composable, or service function. Check here before writing a new one.

| Name                            | File                              | What It Manages / Returns                        |
|---------------------------------|-----------------------------------|---------------------------------------------------|
| `useHookName()`                 | `hooks/use-hook-name.ts`          | [data fetched, side effects, realtime listeners]  |

---

## Component Inventory
> All non-trivial components grouped by folder. Check here before creating a new one.

**`components/[folder]/`**
- `ComponentName.tsx` — [what it renders, key props]

---

## Environment Variables
> Every env var the project uses. Never hardcode these.

| Variable                        | Used In                          | What It Enables                                  |
|---------------------------------|----------------------------------|---------------------------------------------------|
| `ENV_VAR_NAME`                  | `lib/file.ts`                    | [service or feature]                              |

---

## Dev Commands
> All commands needed to work on this project.

| Command           | What It Does                                              |
|-------------------|-----------------------------------------------------------|
| `[cmd]`           | [description]                                             |

---

## External Integrations & Data Contracts
> External systems that read/write this project's data. Document the exact table/field contract.

**[System Name]** (e.g. n8n, Zapier, third-party webhook)
- Writes to: `table_name` — fields: `col1`, `col2`, `col3`
- Reads from: `table_name` — via [method]
- Triggered by: [event or schedule]
- Owner: [who controls this system]

---

## Systems
> Cross-cutting infrastructure and services wired into the project.

| System          | Status       | Details                                           |
|-----------------|--------------|---------------------------------------------------|
| Authentication  | ✅ Active     | e.g. JWT via NextAuth, Google OAuth               |
| Database        | ✅ Active     | e.g. PostgreSQL via Prisma ORM                    |
| Email           | ✅ Active     | e.g. Resend, transactional only                   |
| Payments        | ⚠️ Partial   | e.g. Stripe checkout, webhooks not yet handled    |
| File Storage    | ❌ Not set up | —                                                 |
| Background Jobs | ❌ Not set up | —                                                 |
| Realtime        | ❌ Not set up | —                                                 |
| AI / LLM        | ✅ Active     | e.g. OpenAI GPT-4o via Vercel AI SDK              |

---

## Features
> User-facing features and modules detected in the codebase.

- **[Feature Name]** — [what it does, which files/routes power it] *(added: [date])*
- ...

---

## Workflows
> Multi-step flows that span multiple files, routes, or systems.

**[Workflow Name]** *(e.g. User Registration)*
1. Step one — `file/or/route`
2. Step two — `file/or/route`
3. Step three — `file/or/route`

**[Workflow Name]** *(e.g. Payment Flow)*
1. ...

---

## Removed
- ~~[Feature, system, or module]~~ — [why it was removed] *(removed: [date])*
- ...

---

## Fixed
- [Bug description] → [how it was fixed] *(fixed: [date])*
- ...

---

## Known Issues / TODOs
- [ ] [Issue or task — be specific]
- [ ] ...

---

## Decisions & Notes
> Important architectural or design decisions.

- [Decision] — [Reason/context]
- ...

---

## Session Log
| Date       | Summary                                         |
|------------|-------------------------------------------------|
| [Date]     | [What was done this session in 1–2 sentences]   |

```

---

## Rules

1. **Never overwrite history** — always append, never delete past entries
2. **Keep it factual** — no fluff, no summaries of summaries
3. **Dates matter** — always tag entries with the date they happened
4. **One file per project** — it lives at the project root as `PROJECT_KNOWLEDGE.md`
5. **At session end** — if the user says they're done, offer to update the file before closing
6. **On session start** — if the file exists and user hasn't mentioned it, ask: "Want me to load the project knowledge file so I'm up to speed?"
7. **Check before creating** — before writing any new action, hook, or component, check the inventory sections first. Never duplicate.
8. **Inventory is always current** — every new action/hook/component/env var added in a session must be appended to the relevant inventory section before the session closes.
9. **DB schema is sacred** — any column, table, or relation change must be reflected in `## Database Schema` immediately. An AI working with wrong schema assumptions causes data bugs.
10. **External contracts are explicit** — if an external system (webhook, n8n, third-party) writes to or reads from the DB, document the exact fields in `## External Integrations & Data Contracts`. Never guess the shape.
