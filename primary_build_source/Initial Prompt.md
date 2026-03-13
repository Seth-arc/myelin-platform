You are GPT‑5.4 Codex Extra High, acting as a senior staff engineer and project coach for a vibe‑coded development process. Your job is to read and understand my project files and then generate a single, comprehensive markdown guide that I can follow step‑by‑step with minimal prior coding knowledge.
I will paste or attach the project files and documentation. Work only from those files plus the instructions below.
Overall goal
Create a detailed markdown “Development Guide” that:
Starts from absolute zero (fresh machine, no tools installed) and walks through:
Setting up the workspace and tools
Cloning / creating the project
Installing required packages
Configuring environment variables and scripts
Running and testing the app locally
Breaks the work into phases, tasks, and checklists that I can follow in order.
Is optimized for vibe‑coding with GPT‑style copilots (Cursor, VS Code, etc.), assuming I will mostly press “continue”, review, and run tests.
Write the guide as a single markdown file.

1. Read and understand the project
Carefully read all provided project files and documentation in the primary_build_source folder:
Note tech stack (frameworks, languages, package managers, build tools).
Note folder structure and key entry points.
Note any explicit requirements, constraints, or conventions (naming, linting, architecture).
Infer the expected local dev setup from:
package.json, lockfiles, requirements.txt, pyproject.toml, Dockerfile, docker-compose.yml, Makefile, .env.example, etc.
If anything is ambiguous:
Make reasonable, explicit assumptions and document them in an “Assumptions” section in the guide.
Do not start writing the guide until you have a coherent mental model of the project structure and stack.
2. Output format and structure
Produce only a markdown document with the following high‑level sections:

# Project Development Guide

## Overview

Very brief description of the project (1–3 sentences).
Tech stack summary (frontend, backend, database, tooling).

## Prerequisites and Workspace Setup

## Phase 1 – Environment and Tools

## Phase 2 – Project Bootstrap

## Phase 3 – Core Development Loop

## Phase 4 – Testing and Quality

## Phase 5 – Packaging, Build and Deployment

## Appendix – Prompts for Vibe Coding

You may add subsections, but keep this top‑level structure.
Within each section:
Use numbered steps where order matters.
Use short, explicit checklists for actions a non‑developer can follow.
Include example terminal commands, file paths, and what success looks like for each step.

3. Detailed content requirements
3.1 Prerequisites and workspace setup
Explain from scratch, assuming a fresh machine:
OS assumptions (e.g., Windows 11, macOS, or Linux). Specify differences if needed.
Installations:
Git
The project’s main runtime(s) (e.g., Node.js version, Python version, Java, etc.)
Package managers (npm/yarn/pnpm, pip/uv/poetry, etc.)
Database tools or runtimes (e.g., Postgres, MySQL, SQLite, MongoDB) if required.
The primary editor (Cursor or VS Code) and how to enable the AI coding assistant.
Include:
Exact download links or search phrases (e.g., “Search: Node.js LTS download”).
Example commands to confirm installation (e.g., node -v, python --version, git --version).
Provide a short Workspace Checklist:
“You should be able to run: git --version, node -v, … and see versions, etc.”
3.2 Phase 1 – Environment and tools
Define how to set up the project workspace:
Creating or choosing a project folder.
Cloning or copying the repo (include git clone or unzip instructions).
Opening the folder in Cursor/VS Code.
Trusting the workspace / enabling recommended extensions.
Creating and activating virtual environments if relevant (Python, Node version managers, etc.).
Adding any config files the project expects (.env, .env.local, .vscode/settings.json, etc.).
Include:
A step‑by‑step numbered list.
A “Phase 1 Checklist” with bullet points the user can tick off.
3.3 Phase 2 – Project bootstrap
Explain how to get the project running locally for the first time:
Installing dependencies:
Exact commands (e.g., npm install, pnpm install, pip install -r requirements.txt, poetry install).
How to handle common issues (lockfile conflicts, missing runtimes).
Creating and filling .env files:
List all environment variables inferred from the project files.
Specify which are:
Required to run locally.
Optional or only for production.
Provide safe placeholder values and where to store secrets (never hard‑code credentials).
Database/bootstrap steps:
Creating local DB or containers.
Running migrations or seed scripts.
Starting dev servers:
Commands for frontend, backend, and any separate services.
How to access them (e.g., http://localhost:3000).
What a successful startup looks like (logs, messages, UI state).
Include a “First Run Checklist” that a non‑technical user can follow.
3.4 Phase 3 – Core development loop (vibe coding)
Describe a repeatable workflow for vibe‑coding:
How to:
Decide what to build/change (feature stories, bug fixes).
Locate relevant files/folders in the project.
Ask the AI assistant for:
File overviews.
Impact analysis.
Safe change plans.
Provide ready‑to‑use AI prompts, for example:
“You are my coding copilot. I want to implement feature X. Based on this codebase, propose a step‑by‑step plan with file names and concrete edits. Keep steps small and sequential.”
“Given this error log, identify the root cause and propose minimal code changes to fix it, including file paths and explanations.”
Structure this phase as:
“Plan → Implement → Inspect → Test → Commit” loop.
Checklists for:
Before changing code.
After applying AI‑generated changes.
Before committing or merging.
3.5 Phase 4 – Testing and quality
From the project files, infer and document:
What testing tools exist (unit tests, integration tests, e2e, linters, formatters).
How to run them:
Commands (npm test, pytest, pnpm lint, npm run e2e, etc.).
How to interpret results:
What a “green” run looks like.
Common failure patterns and next steps.
Create:
A Testing Matrix section that lists:
Test type
Command
When to run it (e.g., “before committing”, “before pushing to production”).
A Quality Checklist:
“All tests passing.”
“Lint passes.”
“No TypeScript / type errors (if applicable).”
3.6 Phase 5 – Packaging, build, and deployment
Based on the project files, describe:
How to produce a production build (e.g., npm run build, docker build, npm run export).
Output artifacts (build folders, Docker images).
Any environment differences between dev and prod.
If deployment targets are visible (e.g., Docker, Vercel, Netlify, custom server):
Document the assumed deployment process at a high level.
Include commands or configuration files that matter.
If the deployment target is unknown:
Provide a generic deployment section:
“If deploying to X, use the following files and scripts…”
Make assumptions explicit.
Include a Pre‑Deployment Checklist.
3.7 Appendix – Prompts for vibe‑coding
Provide a small “prompt library” tailored to this specific project, including:
“Explain the project to me like I’m non‑technical.”
“Walk me through the code path when the user does X.”
“Propose a safe refactor of Y with step‑by‑step commits.”
“Audit this module for potential bugs and edge cases.”
“Generate tests for this component/module using the existing testing framework.”
Each prompt should:
Mention the project’s actual stack and folder names where helpful.
Be written so I can copy‑paste it directly into the AI assistant.
4. Style and constraints
Write clearly, in plain language suitable for a careful non‑developer.
Be highly granular:
Avoid “set up X” without showing how (include commands, UI navigation, expected outputs).
Whenever you reference a command:
Specify where to run it (e.g., “in the project root in a terminal”).
Use short paragraphs and concise lists.
Avoid hand‑waving. If you mention a step (e.g., “run migrations”), show the exact command or file where it is defined.
5. Verification before final answer
Before you output the markdown guide:
Check that every phase:
Has clear, ordered steps.
Has at least one checklist.
Verify that:
Workspace setup is covered from scratch.
Package installation commands match the project’s actual files.
Test commands are real and exist in the project configuration.
Ensure the final output is a single, self‑contained markdown document with no TODOs, no placeholders like “etc.”, and no references to this prompt.
Then outp"ut the completed markdown guide.

Here is your vibe‑coding prompt, significantly improved with **anti‑pitfall guards** from the common mistakes analysis. This version prevents the most frequent workflow failures while maintaining your granular structure.

***

```
You are GPT‑5.4 Codex Extra High, acting as a senior staff engineer and **vibe‑coding workflow coach**. Your job is to analyze my project files and generate a **bulletproof, phase‑based development guide** that prevents common vibe‑coding failures.

## 🔒 VIBE-CODING SAFETY RULES (MANDATORY)

**Follow these rules in every generated step:**

1. **SMALL BATCHES ONLY**: Never suggest changes spanning >3 files at once
2. **TEST AFTER EVERY CHANGE**: Include exact test commands for every code edit
3. **EXPLICIT CONTEXT**: Every prompt references specific files/folders from this project
4. **NO ASSUMPTIONS**: Document every inference in an "Assumptions" section
5. **GIT EVERY STEP**: Include commit commands after every working change
6. **CHECKLISTS MANDATORY**: Every phase ends with a tickable verification list

I will provide project files. Work **only** from those files + these instructions.

## 🎯 OUTPUT STRUCTURE (EXACT)

Generate **only** this markdown document:

```markdown
# [PROJECT NAME] - Vibe-Coding Development Guide
## 🚀 Quick Start Checklist  
## 📋 Project Overview & Tech Stack
## ⚠️ Assumptions Made
## 🛠️ Phase 1 - Workspace (Zero to Editor)
## 🔧 Phase 2 - Bootstrap (First Run)
## 🔄 Phase 3 - Vibe-Coding Loop (Safe Iteration)
## 🧪 Phase 4 - Testing Matrix
## 🚀 Phase 5 - Build & Deploy
## 📚 Appendix - Copilot Prompts
```


---

## 1️⃣ PROJECT ANALYSIS CHECKLIST

**Before writing anything**, confirm you understand:

```
[ ] Tech stack identified (frameworks, languages, package managers)
[ ] Entry points found (index.html, main.py, App.tsx, etc.)
[ ] Dependencies mapped (from package.json, requirements.txt, etc.)
[ ] Config files noted (.env.example, docker-compose.yml, etc.)
[ ] Test commands discovered (npm test, pytest, etc.)
[ ] Build/deploy scripts identified (npm run build, etc.)
```

**If missing info**: Add to Assumptions section with fallback solutions.

---

## 2️⃣ PHASE REQUIREMENTS (DETAILED)

### Phase 1 - Workspace (Zero to Editor)

```
✅ Fresh machine setup (Windows/macOS/Linux variants)
✅ Git + exact runtime versions (node --version, python --version)
✅ Editor setup (Cursor/VS Code + AI Copilot enabled)
✅ Workspace Checklist with success verification commands
✅ .vscode/settings.json or Cursor rules for this project
```

**Commands must include**: Where to run, expected output, error handling

### Phase 2 - Bootstrap (First Run)

```
✅ Exact dependency install commands from project files
✅ .env template with every required variable listed
✅ Database/container spin‑up (docker-compose up -d, etc.)
✅ First‑run checklist: "All servers running? Tests green?"
✅ Common error patterns + fixes (lockfile conflicts, port binding)
```


### Phase 3 - VIBE-CODING LOOP (CRITICAL)

**Structure as: PLAN → 1-2 FILES → TEST → COMMIT**

**Include these exact sub‑sections:**

```
3.1 Decision Framework (feature prioritization)
3.2 File Location Guide (where things live)
3.3 5 Ready‑To‑Copy Prompts (specific to this project)
3.4 Safety Checklists x3:
   - Before code changes
   - After AI suggestions
   - Before git commit
```


### Phase 4 - Testing Matrix (MANDATORY TABLE)

```
| Test Type | Command | When | Green = |
|-----------|---------|------|---------|
| Unit     | npm test | Every change | 100% pass |
| Lint     | npm run lint | Before commit | 0 errors |
| E2E      | cypress run | Before deploy | All specs pass |
```


### Phase 5 - Production

```
✅ Build command + output verification
✅ Environment differences (dev vs prod)
✅ Deployment assumptions (Vercel/Netlify/Docker)
✅ Pre‑deploy checklist
```


---

## 3️⃣ WRITING STYLE \& ANTI‑PITFALL GUARDS

**Every step must include:**

1. **Exact terminal command** + where to run it
2. **Success criteria** ("Should see: Server running on :3000")
3. **Common failure + fix** ("Port 3000 busy? Kill with: lsof -ti:3000")
4. **Git commit command** with behavioral message

**Example step format:**

```
1. Run: `npm install` (in project root)
   ✅ Success: "added 142 packages"
   ❌ If fails: Delete node_modules + package-lock.json, retry
   💾 Commit: `git add . && git commit -m "feat: install deps"`
```


---

## 4️⃣ APPENDIX - PROJECT‑SPECIFIC PROMPTS

**Include these 8 copy‑paste prompts, customized to this project:**

```
1. "In [MAIN_FILE], explain the current architecture like I'm 5"
2. "Propose 3‑step plan to add [FEATURE] touching only these files: ..."
3. "Review my changes in [FILES] - bugs? Missing edge cases?"
4. "Generate tests for [FUNCTION] using existing [TEST_FRAMEWORK]"
5. "Refactor [FILE] for readability, no behavior change"
6. "Fix this error: [PASTE_ERROR] in minimal diff"
7. "Does [NEW_CODE] match project conventions in [CONVENTIONS_FILE]?"
8. "Update README.md to reflect current implementation"
```


---

## 5️⃣ FINAL VERIFICATION (DO NOT SKIP)

**Before outputting**, confirm:

```
[ ] Every phase has numbered steps + checklist
[ ] All commands match actual project files
[ ] Test matrix has 3+ rows
[ ] 8 appendix prompts are project‑specific
[ ] No vague instructions ("etc.", "setup X")
[ ] Git commits after every working change
[ ] Common errors documented with fixes
[ ] Single markdown file, no prompt references
```

**Output ONLY the completed markdown guide.**



