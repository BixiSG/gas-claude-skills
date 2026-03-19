---
name: gas-workflow
description: >
  Workflow for creating, storing, editing, and deploying Google Apps Script (GAS)
  projects. Handles folder setup, local/GitHub storage decisions, clasp CLI
  integration, and push-with-notes discipline. Use this skill whenever the user
  mentions Google Apps Script, GAS, Apps Script, clasp, .gs files, script editor,
  Sheets macros, Docs add-ons, or any project that lives on script.google.com —
  even if they only say "I want to automate something in Google Sheets" or "write
  me an Apps Script". If Apps Script is involved at all, use this skill.
---

# GAS Workflow

A structured workflow for Google Apps Script projects that keeps code organized,
version-controlled, and deployable via clasp. Every GAS session flows through
four phases: **Create → Store → Edit → Push**. Each phase has decision points
where you ask the user before proceeding — never assume defaults silently.

---

## Prerequisites Check

Before starting any GAS work, verify the toolchain is available. Run these
checks silently and only surface problems:

```bash
node --version   # Need v20+
npm --version
clasp --version  # If missing, offer: npm install -g @google/clasp
git --version
gh --version     # For GitHub operations
```

If `clasp` is not installed, ask:

> clasp (the Apps Script CLI) isn't installed. Want me to install it?
> `npm install -g @google/clasp`

If clasp is installed but the user has never logged in (`~/.clasprc.json`
doesn't exist), prompt:

> You'll need to authorize clasp with your Google account. Run `clasp login`
> in your terminal — it'll open a browser window for you to sign in. Let me
> know when you're done.

**Do not run `clasp login` yourself** — it requires browser-based OAuth that
the user must complete interactively.

---

## Phase 1: Create

**Goal:** Set up the project folder and initialize the GAS project.

### Step 1.1 — Ask about folder structure

> Do you want a dedicated folder for this project, or should I put the files
> in the current directory?

If dedicated folder: create it with a sensible name based on the project
description (e.g., `gas-invoice-generator/`). Use kebab-case.

### Step 1.2 — Ask about project type

> What type of Apps Script project is this?
>
> 1. **Standalone** — lives on its own in Drive
> 2. **Bound to Sheets** — attached to a specific Google Sheet
> 3. **Bound to Docs** — attached to a Google Doc
> 4. **Bound to Slides** — attached to a Google Slides deck
> 5. **Bound to Forms** — attached to a Google Form
> 6. **Web App** — deployed as a web application
> 7. **API executable** — called from external apps

Map answers to clasp types: `standalone`, `sheets`, `docs`, `slides`, `forms`,
`webapp`, `api`.

### Step 1.3 — Initialize with clasp or scaffold locally

**If the user has an existing script** they want to bring under version control:

```bash
clasp clone <scriptId> --rootDir .
```

**If creating a new project:**

```bash
clasp create --title "Project Name" --type <type> --rootDir .
```

**If clasp isn't set up yet** (user skipped login, or prefers local-first):
scaffold the files manually and defer clasp connection to Phase 3.

Minimum scaffold:

```
project-name/
├── appsscript.json       # Manifest
├── Code.gs               # Main entry point
└── README.md             # Brief project description (optional)
```

Default `appsscript.json`:

```json
{
  "timeZone": "America/New_York",
  "dependencies": {},
  "exceptionLogging": "STACKDRIVER",
  "runtimeVersion": "V8"
}
```

Ask the user what timezone they prefer if not obvious from context.

---

## Phase 2: Store

**Goal:** Decide where code lives and set up version control.

### Step 2.1 — Ask storage preference

> How do you want to store this project?
>
> 1. **Local only** — files stay on your machine, no remote backup
> 2. **Private GitHub repo only** — code lives on GitHub, pulled locally when needed
> 3. **Both local + private GitHub repo** — local working copy synced to GitHub

### Step 2.2 — If GitHub is involved (options 2 or 3)

#### 2.2a — Ask who pushes

> For pushing to GitHub, do you want:
>
> 1. **Me (Claude) to push** — I'll use `gh` CLI to create the repo and push commits
> 2. **You to push** — I'll prepare everything and give you the commands to run

#### 2.2b — If Claude pushes: create the repo

```bash
gh repo create <project-name> --private --source . --remote origin
git init
git add -A
git commit -m "Initial commit: scaffold GAS project

- Created project structure with clasp
- Added appsscript.json manifest
- Added Code.gs entry point"
git branch -M main
git push -u origin main
```

#### 2.2c — If user pushes: check GitHub connection

Check if the user has `gh` authenticated:

```bash
gh auth status
```

If not authenticated, walk them through it:

> You'll need to connect to GitHub. The easiest way:
>
> ```bash
> gh auth login
> ```
>
> This will walk you through browser-based login. Let me know when you're done.

Then provide the exact commands for them to run, nicely formatted and
copy-pasteable.

### Step 2.3 — Set up .gitignore

Always create a `.gitignore` appropriate for GAS projects:

```
# clasp
.clasprc.json
node_modules/

# OS
.DS_Store
Thumbs.db

# Editor
.vscode/
.idea/
*.swp
```

---

## Phase 3: Edit / Update

**Goal:** When the user modifies code, keep everything in sync.

### Step 3.1 — After any code edit, ask about sync

After writing or modifying any `.gs`, `.js`, or `appsscript.json` file, prompt:

> I've updated the code. Want me to:
>
> 1. **Push to Apps Script via clasp** — syncs to script.google.com
> 2. **Push to GitHub** — commits and pushes to your repo
> 3. **Both** — clasp push + git commit & push
> 4. **Neither for now** — just keep the local changes

This question is the heartbeat of the workflow. Ask it every time code changes,
but be smart about batching — if the user is making rapid iterative changes
("try X", "no wait, change Y"), don't ask after every single edit. Wait for a
natural pause or when they say something like "ok that looks good" or "let's
save this".

### Step 3.2 — clasp push

If clasp pushing, check that `.clasp.json` exists and has a valid `scriptId`:

```bash
cat .clasp.json 2>/dev/null
```

**If `.clasp.json` is missing or has no scriptId:**

> I need a Script ID to push via clasp. You can find it by:
>
> 1. Opening your Apps Script project at script.google.com
> 2. Going to Project Settings (gear icon)
> 3. Copying the Script ID
>
> Or give me the URL to your script and I'll extract it.

Then connect:

```bash
echo '{"scriptId":"<SCRIPT_ID>","rootDir":"."}' > .clasp.json
```

**If clasp has never been used in this project**, walk through first-time setup:

> Let's connect this project to Apps Script. I'll need your Script ID.
> If this is a brand new project, I can create one with `clasp create`.

Then push:

```bash
clasp push
```

If `clasp push` fails with auth errors, remind the user to run `clasp login`.

### Step 3.3 — GitHub push (always with update notes)

**Every GitHub push gets a short, descriptive commit message.** This is
non-negotiable — no generic "update" messages. The commit message should
describe what actually changed.

Format:

```
<type>(<scope>): <short description>

<optional body with more detail>
```

Types: `feat`, `fix`, `refactor`, `docs`, `style`, `chore`

Examples:
- `feat(sheets): add auto-formatting for invoice rows`
- `fix(trigger): correct timezone offset in daily trigger`
- `refactor(utils): extract date helpers into separate file`

```bash
git add -A
git commit -m "<commit message>"
git push
```

**Before committing**, always show the user what's being committed:

> Here's what I'm about to commit:
>
> **Changed files:** [list them]
> **Commit message:** `feat(sheets): add conditional formatting rules`
>
> Look good?

---

## Phase 4: Push Discipline

**Goal:** Every push (clasp or git) is intentional and documented.

### Rules (always enforced)

1. **No silent pushes.** Always confirm with the user before pushing anywhere.
2. **Every git commit has meaningful notes.** Use conventional commit format.
3. **clasp push = production push.** Remind the user that `clasp push`
   overwrites what's on script.google.com. If they have collaborators editing
   in the browser, warn about conflicts.
4. **Suggest `clasp pull` before `clasp push`** if the project might have been
   edited in the browser since the last sync. This prevents overwriting
   browser-side changes.

### Push checklist (run mentally before every push)

- [ ] Code has been tested or reviewed
- [ ] Commit message describes the change
- [ ] No secrets or API keys in the code
- [ ] `.clasp.json` is in `.gitignore` (contains scriptId)
- [ ] If clasp pushing: user is aware this overwrites the remote script

---

## Quick Reference: clasp Commands

| Command | What it does |
|---------|-------------|
| `clasp login` | Authenticate with Google (browser OAuth) |
| `clasp logout` | Clear stored credentials |
| `clasp create --title "X" --type sheets` | New project |
| `clasp clone <scriptId>` | Download existing project |
| `clasp pull` | Download remote → local |
| `clasp push` | Upload local → remote (overwrites!) |
| `clasp push --watch` | Auto-push on file changes |
| `clasp open` | Open in Apps Script editor |
| `clasp deploy` | Create a versioned deployment |
| `clasp deployments` | List all deployments |
| `clasp version "description"` | Create an immutable version |

---

## Edge Cases

### User wants TypeScript
clasp supports TypeScript natively. If the user prefers TS:
- Use `.ts` files instead of `.gs`
- clasp automatically transpiles on push
- Add a `tsconfig.json` if the project needs custom TS settings

### User wants to work on an existing project
Skip Phase 1 creation. Instead:
1. Get the Script ID from the user
2. `clasp clone <scriptId>`
3. Pick up at Phase 2 (storage decisions)

### Multiple environments (dev/staging/prod)
Suggest using clasp deployments for this:
- Development: push directly, test in editor
- Staging: `clasp version "v1.2-rc1"` then `clasp deploy`
- Production: deploy a tested version

### User doesn't want clasp at all
That's fine. Skip all clasp steps, work purely with local files + GitHub.
The user can copy-paste into the script editor manually. Still enforce
git discipline and commit messages.
