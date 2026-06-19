# ytBackend - Professional Project Setup

> This lesson moves past basic Express/Mongoose syntax and into **professional, production-grade project setup** — the kind of structure, tooling, and conventions used in real industry codebases.

---

## 1. Overview

This lesson kicks off the **professional journey** of the backend series. Up to this point, the series covered only the basics — a short intro to Express, a short intro to Mongoose. From here onward, the goal is to build a **production-grade backend** for a YouTube-like platform, following the exact conventions that real companies use.

Rather than jumping straight into writing model code or routes, this file focuses entirely on **project scaffolding**: initializing the repository correctly, setting up Git the professional way, designing a clean folder structure, and configuring developer tooling (`nodemon`, `Prettier`) so that a team can collaborate without friction.

**Practical purpose:** By the end of this lesson, you'll have a fully scaffolded Node.js backend project — version-controlled, properly structured, and configured with consistent code formatting — ready for actual feature code (models, controllers, routes) to be added in subsequent lessons.

---

## 2. Key Concepts

### 2.1 Why Project Setup Matters
Professional backend development isn't just about writing logic — it's about creating a codebase that:
- Multiple developers can work on without conflicts
- Follows industry-standard folder conventions
- Keeps secrets and environment-specific values out of version control
- Enforces consistent code style across contributors

### 2.2 Git Repository Initialization
A fresh Git repository is created for this project (separate from earlier basic-concept files) because this is the first project worth tracking properly from end to end.

Key Git concepts covered:
- **`git init`** — initializes a local repository
- **Default branch renaming** — GitHub's standard is `main`, not `master`, so the local branch is renamed to match
- **Remote setup** — connecting the local repo to a GitHub repository
- **Upstream tracking** — using `git push -u origin main` so future pushes don't need the full command

### 2.3 The Empty Folder Problem in Git
**Git does not track empty folders.** It only tracks files. So if you create a folder like `public/temp` for temporary file storage (e.g., during file uploads), Git will silently ignore it until it contains a file.

**Solution:** Create a placeholder file — by convention named **`.gitkeep`** — inside the empty folder. This file is intentionally empty; its only job is to give Git something to track so the folder structure is preserved when pushed.

### 2.4 Environment Variables and `.gitignore`
In professional projects, sensitive data — API keys, secrets, database URLs — must never be committed to version control.

- **`.env`** — stores actual environment variable key-value pairs (real secrets). This file is added to `.gitignore` so it's never pushed.
- **`.env.sample`** — a *template* file showing which variables are needed (with placeholder/empty values), committed to Git so other developers/contributors know what to configure.
- **`.gitignore`** — a file listing patterns/paths that Git should never track. Commonly generated using online **gitignore generators** (templated per tech stack, e.g., Node.js), which automatically exclude `node_modules`, environment files, build artifacts, OS-specific files, etc.

In production, environment variables aren't read from a `.env` file at all — they're injected directly by the hosting platform (AWS, DigitalOcean, etc.) through their dashboard/config system, which is more secure.

### 2.5 Module System: ES Modules vs CommonJS
JavaScript supports two import/export syntaxes:
- **CommonJS** — `require()` / `module.exports` (the older Node.js default)
- **ES Modules (ESM)** — `import` / `export` (modern standard syntax)

This project standardizes on **ES Modules** throughout for consistency. To enable this in Node.js, the `package.json` must declare:
```json
"type": "module"
```

> **Note:** One known friction point — `.env` files traditionally required the CommonJS-style `require('dotenv').config()` syntax rather than the cleaner ESM import. This nuance and its workaround is deferred to a future lesson.

### 2.6 Professional Folder Structure (`src/`)
All source code lives inside a `src/` folder rather than the project root, keeping configuration files (package.json, .env, etc.) separate from application logic.

Inside `src/`, the following standard sub-folders are created:

| Folder | Purpose |
|---|---|
| `controllers/` | Contains the core business logic / request-handling functions |
| `db/` | Database connection logic (MongoDB, PostgreSQL, etc. — connection code lives here regardless of DB type) |
| `middlewares/` | Functions that run *between* an incoming request and the final route handler (e.g., authentication checks, validation) |
| `models/` | Database schema/model definitions |
| `routes/` | Route definitions, separated out instead of being written inline in the main entry file |
| `utils/` | Reusable utility/helper functions used across the app (e.g., file upload helpers, email-sending helpers, token helpers) |

Two top-level files are also created in `src/`:
- `app.js` — Express application configuration
- `index.js` — application entry point
- `constants.js` — for storing fixed constant values used app-wide

> **Why a `utils` folder?** Certain functionality repeats across the app — file uploads (used for profile pictures *and* videos), email sending, token handling, etc. Rather than duplicating this logic, it's written once as a reusable utility and imported wherever needed.

### 2.7 `nodemon` — Auto-Restart Development Tool
Normally, any time you change server code, you must manually stop and restart the Node process to see the change take effect. **`nodemon`** automates this: it watches your files and automatically restarts the server on save.

- Installed as a **dev dependency** (`devDependency`), because it's only needed during development, not in production
- Command: `npm install -D nodemon` (or `--save-dev` / `-D`)

### 2.8 Dependencies vs Dev Dependencies
- **`dependencies`** — packages required for the app to run in production (e.g., Express, Mongoose)
- **`devDependencies`** — packages only needed during development (e.g., `nodemon`, `prettier`). They don't affect the production bundle size or behavior since they're excluded from production deployments.

### 2.9 Prettier — Code Formatting Consistency
When multiple developers work on the same codebase, inconsistent formatting (tabs vs. spaces, semicolons vs. no semicolons, single vs. double quotes) causes **massive merge conflicts** in Git — even when the actual logic hasn't changed.

**Prettier** is a code formatter that enforces a single, agreed-upon style across the entire team. It's configured via:
- **`.prettierrc`** — the configuration file defining formatting rules (quote style, tab width, semicolons, trailing commas, etc.)
- **`.prettierignore`** — tells Prettier which files/folders to *skip* formatting on (critically: `.env` files, since Prettier can corrupt key-value syntax; also `node_modules`, build/distribution folders)

Common `.prettierrc` settings discussed:
| Setting | Value Used | Meaning |
|---|---|---|
| `singleQuote` | `false` | Use double quotes instead of single quotes |
| `bracketSpacing` | `true` | Adds spacing inside object braces, e.g. `{ a: 1 }` |
| `tabWidth` | `2` | Two spaces per indentation level |
| `semi` | `true` | Statements end with semicolons |
| `trailingComma` | `"es5"` | Adds trailing commas where valid in ES5 (arrays, objects) |

### 2.10 Why This Level of Setup Matters
This isn't busywork — it's what separates hobby projects from production-grade software:
- Prevents secrets from leaking into public repositories
- Prevents formatting-related merge conflicts in team environments
- Creates a scalable, navigable folder structure as the project grows
- Establishes patterns (utils, middlewares, separated routes) that scale to large, complex applications

---

## 3. Code Breakdown

### Step 1 — Project Folder & Git Initialization
```bash
mkdir "chai aur backend"
cd "chai aur backend"
git init
```
Creates the project folder and initializes a local Git repository.

### Step 2 — `package.json` Initialization
```bash
npm init
```
Walks through prompts (package name, version, description, entry point, author, license) to generate `package.json`. The entry point is set to `index.js` (though this evolves once the `src/` structure is introduced).

### Step 3 — `README.md` Creation
A `README.md` file is created at the project root containing the project title, a short description, and links (e.g., to a data model diagram created in a design tool). This documents the project for anyone visiting the repository.

### Step 4 — Connecting to GitHub
```bash
git add .
git commit -m "Start: Add initial files for backend"
git branch -M main
git remote add origin <repository-url>
git push -u origin main
```
- `git branch -M main` renames the default branch from `master` to `main`
- `git remote add origin` links the local repo to the GitHub repository
- `git push -u origin main` pushes code and sets the upstream branch, so future pushes can simply use `git push`

### Step 5 — Creating the `public/temp` Folder Structure
```bash
mkdir -p public/temp
touch public/temp/.gitkeep
```
This folder will later be used for temporarily storing uploaded files (e.g., images) before they're sent to third-party cloud storage. Since the folder starts empty, a `.gitkeep` file is added so Git tracks the (otherwise empty) directory.

### Step 6 — `.gitignore` and Environment Files
```bash
touch .env
touch .env.sample
```
A `.gitignore` file is generated (typically via an online generator for Node.js projects) and includes entries like:
```
node_modules
.env
.env.test
.env.production
```
`.env` holds real secrets and is excluded from Git. `.env.sample` is a template (committed to Git) showing which environment variables the project expects, without real values.

### Step 7 — Enabling ES Modules
In `package.json`:
```json
{
  "type": "module"
}
```
This switches the project from CommonJS (`require`) to ES Module (`import`) syntax throughout.

### Step 8 — Building the `src/` Structure
```bash
mkdir src
cd src
touch app.js constants.js index.js
mkdir controllers db middlewares models routes utils
```
Each new folder (controllers, db, middlewares, models, routes, utils) starts empty, so — as with `public/temp` — a `.gitkeep` placeholder would be needed in each to track them in Git until real files are added.

### Step 9 — Adding the `dev` Script with Nodemon
```bash
npm install -D nodemon
```
In `package.json`:
```json
"scripts": {
  "dev": "nodemon src/index.js"
}
```
Running `npm run dev` now starts the server via `nodemon`, which watches `src/index.js` (and its dependencies) and restarts automatically on file changes.

### Step 10 — Installing and Configuring Prettier
```bash
npm install -D prettier
```
Two configuration files are created at the project root:

**`.prettierrc`**
```json
{
  "singleQuote": false,
  "bracketSpacing": true,
  "tabWidth": 2,
  "semi": true,
  "trailingComma": "es5"
}
```

**`.prettierignore`**
```
node_modules
.env
.env.sample
dist
```

### Step 11 — Commit and Push the Full Setup
```bash
git add .
git commit -m "Add prettier config"
git push
```
All scaffolding — folder structure, Git config, Prettier rules, package.json scripts — is pushed to GitHub.

### Flow Summary
```
Create folder → git init → npm init → README.md
       ↓
Connect to GitHub (remote + push)
       ↓
Set up public/temp (+ .gitkeep) for temp file storage
       ↓
Set up .gitignore, .env, .env.sample
       ↓
Enable ES Modules in package.json
       ↓
Build src/ structure (controllers, db, middlewares, models, routes, utils)
       ↓
Install nodemon → add "dev" script
       ↓
Install & configure Prettier (.prettierrc, .prettierignore)
       ↓
Commit and push final setup
```

---

## 4. Functions & Methods

> Note: This lesson is primarily CLI/tooling-focused. The "functions" here are Git, npm, and Node CLI commands rather than JavaScript functions — documented in the same problem/syntax/parameters/return format for consistency.

### `git init`
**Problem it solves:** Starts tracking a project directory as a Git repository.
**Syntax:**
```bash
git init
```
**Parameters:** None required.
**Return value:** Creates a hidden `.git/` folder in the current directory; no terminal output value, but initializes version control.

---

### `npm init`
**Problem it solves:** Generates a `package.json` file describing the project (name, version, dependencies, scripts) — required for any Node.js project.
**Syntax:**
```bash
npm init [-y]
```
**Parameters:**
- `-y` *(optional flag)* — skips the interactive prompts and accepts all defaults.

**Return value:** Creates a `package.json` file in the current directory.

---

### `git branch -M <branch-name>`
**Problem it solves:** Renames the current branch (used here to rename `master` to `main`, matching GitHub's modern default).
**Syntax:**
```bash
git branch -M <new-branch-name>
```
**Parameters:**
- `<new-branch-name>` *(string)* — the new name for the current branch, e.g. `"main"`.

**Return value:** No output; the active branch is renamed.

---

### `git remote add <name> <url>`
**Problem it solves:** Links your local repository to a remote repository (e.g., on GitHub) so you can push/pull code.
**Syntax:**
```bash
git remote add <name> <url>
```
**Parameters:**
- `<name>` *(string)* — typically `"origin"`, the conventional name for the primary remote.
- `<url>` *(string)* — the remote repository's URL, e.g. `https://github.com/username/repo.git`.

**Return value:** No output; registers the remote for future push/pull commands.

---

### `git push -u <remote> <branch>`
**Problem it solves:** Uploads local commits to the remote repository and sets up tracking so future pushes don't require specifying the remote/branch again.
**Syntax:**
```bash
git push -u <remote> <branch>
```
**Parameters:**
- `-u` *(flag)* — sets the upstream branch for tracking.
- `<remote>` *(string)* — e.g., `"origin"`.
- `<branch>` *(string)* — e.g., `"main"`.

**Return value:** No return value in code terms; subsequent commands can simply be `git push`.

---

### `npm install -D <package>` (alias: `--save-dev`)
**Problem it solves:** Installs a package as a **development dependency** — needed only while coding, not in production.
**Syntax:**
```bash
npm install -D <package-name>
```
**Parameters:**
- `<package-name>` *(string)* — name of the npm package, e.g. `"nodemon"`, `"prettier"`.
- `-D` / `--save-dev` *(flag)* — marks it under `devDependencies` in `package.json` rather than `dependencies`.

**Return value:** Installs the package into `node_modules/` and adds an entry to `package.json` under `devDependencies`.

---

### `nodemon <entry-file>`
**Problem it solves:** Eliminates the need to manually stop/restart the Node server every time code changes — it auto-restarts on file save.
**Syntax:**
```bash
nodemon <path-to-entry-file>
```
**Parameters:**
- `<path-to-entry-file>` *(string/path)* — the file to run and watch, e.g. `"src/index.js"`.

**Return value:** Starts a long-running process that restarts the server automatically whenever a watched file changes.

---

### `mkdir` / `touch` (used for `.gitkeep`)
**Problem it solves:** Git does not track empty directories. Creating a placeholder file inside an otherwise-empty folder ensures Git registers and preserves that folder structure.
**Syntax:**
```bash
mkdir <folder-name>
touch <folder-name>/.gitkeep
```
**Parameters:**
- `<folder-name>` *(string)* — name of the folder to create, e.g. `"public/temp"`.
- `.gitkeep` — conventional (not Git-enforced) filename used purely as a tracked placeholder; it's intentionally empty.

**Return value:** Creates an empty file that, once committed, forces Git to keep the parent folder under version control.

---

## 5. Logic Explanation

The overall logic of this setup process follows a deliberate, layered sequence:

1. **Establish version control first.** Before any code is written, the project is wired up to Git and GitHub. This ensures every subsequent step — folder creation, configuration, dependency installs — is captured in history from the very beginning.

2. **Solve the "empty folder" problem early.** Because Git only tracks files (not folders), any folder meant to exist but start empty (like `public/temp` for temporary uploads, or `src/controllers`) needs a `.gitkeep` file. This is addressed as soon as such folders are introduced, so the intended structure survives a push to GitHub.

3. **Separate secrets from code.** `.env` is immediately excluded via `.gitignore`, while `.env.sample` is committed instead — so the *shape* of required configuration is documented without ever exposing real values. This logic anticipates the project eventually being deployed, where environment variables get sourced from the hosting platform instead of a file.

4. **Standardize the module system before writing any logic.** Setting `"type": "module"` in `package.json` happens before any actual application code, ensuring all future files consistently use `import`/`export` syntax rather than mixing `require()` and `import`.

5. **Design the folder architecture around responsibility, not convenience.** Each `src/` subfolder (`controllers`, `db`, `middlewares`, `models`, `routes`, `utils`) maps to a distinct responsibility in a typical backend request lifecycle:
   - A request arrives → may pass through `middlewares` (e.g., auth checks)
   - It's routed via `routes`
   - Handled by logic in `controllers`
   - Which may interact with `models` (schema/data) connected through `db`
   - And may call shared logic from `utils` (e.g., file upload, email, tokens)

   This separation means as the project scales to many features, related code stays grouped and easy to locate.

6. **Automate repetitive developer tasks.** `nodemon` removes the manual restart-the-server step, and `Prettier` removes manual, error-prone formatting decisions — both addressing friction that would otherwise compound as the codebase and team grow.

7. **Lock in team-wide consistency before collaboration begins.** Prettier's configuration is committed to the repo (not just installed locally) so that *every* contributor — regardless of personal editor settings — produces identically formatted code, preventing formatting-driven merge conflicts.

---

## 6. Problem It Solves

**The core problem:** Jumping straight into writing backend logic (models, routes, controllers) without first establishing proper project hygiene leads to:
- Leaked secrets and credentials in public repositories
- Messy, unscalable folder structures as features pile up
- Painful Git merge conflicts caused by inconsistent code formatting across team members
- Manual, repetitive developer workflow friction (restarting servers by hand)
- Confusion for new contributors who don't know where different types of logic belong

**How this setup solves it:**
- **Git + `.gitignore` + `.env`/`.env.sample`** → secrets never reach version control, while configuration requirements are still documented for collaborators.
- **`.gitkeep`** → ensures the intended folder architecture is preserved in the repository even before real files exist in each folder.
- **The `src/` folder structure** (`controllers`, `db`, `middlewares`, `models`, `routes`, `utils`) → gives every piece of future code an unambiguous, conventional home, matching what's used across the industry (with only minor naming variations between companies).
- **ES Modules standardization** → avoids inconsistent import/export styles across the codebase.
- **`nodemon`** → removes manual restart overhead during development.
- **Prettier + `.prettierrc` + `.prettierignore`** → guarantees consistent code style across all contributors, drastically reducing formatting-related merge conflicts — critical once multiple developers (or AI tools) are committing to the same repository.

In short: this lesson solves the problem of **starting a backend project the way professional engineering teams do**, so that as real features (authentication, video uploads, comments, playlists, etc.) are added in future lessons, the foundation is already clean, secure, and scalable.

---

## What's Next

Future lessons in this series will cover:
- Professional error handling and standardized API response formats
- Writing reusable, modular utility code (e.g., file upload helpers)
- Building out the actual data models (User, Video, etc.)
- Database connection logic in `src/db`
- Authentication concepts: tokens, sessions, cookies, password protection

> This README documents the **project setup phase only** — no application logic (models, controllers, routes) has been written yet. That begins in subsequent lessons of the series.