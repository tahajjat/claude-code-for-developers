# Claude Code for Developers

A practical reference repo for software developers learning to work effectively with Claude Code. Each project folder contains real-world examples drawn from production codebases, with context files that teach Claude how the codebase works.

---

## Table of Contents

- [What This Repo Is](#what-this-repo-is)
- [How It Works](#how-it-works)
- [Projects](#projects)
  - [WAMY NGO ERP](#wamy-ngo-erp)
    - [Finance Module](#wamy-ngo--finance-module)
  - [GB Company (Placeholder)](#gb-company)
- [Using CLAUDE.md Files](#using-claudemd-files)
- [Contributing](#contributing)

---

## What This Repo Is

This repo is a teaching resource. Each `projects/` subdirectory represents a real or representative software project. Inside each project you will find:

- A `CLAUDE.md` file that gives Claude Code the context it needs to understand the project stack, architecture, and conventions.
- A `README.md` that explains the project or module for human readers.
- Feature-specific sub-folders with their own `CLAUDE.md` files scoped to that area.

The goal is to show developers how to write good context files so that Claude Code can contribute meaningfully to their specific codebase.

---

## How It Works

When you open a project folder in Claude Code, it automatically reads any `CLAUDE.md` files in the current directory and parent directories. This means:

- Root `CLAUDE.md` → always loaded (repo-wide rules)
- `projects/wamy-ngo/CLAUDE.md` → loaded when working in that project
- `projects/wamy-ngo/finance-module/CLAUDE.md` → loaded when working in that module

Each file narrows the context: stack, naming conventions, architecture decisions, anti-patterns, and open issues.

---

## Projects

### WAMY NGO ERP

> Spring Boot 3.2 · React 19 · MySQL 8 · GitHub Actions · Robi Cloud

A multi-module ERP built for a non-governmental organisation. Covers project-based accounting, HR, procurement, and reporting.

| Module | Status | Path |
|--------|--------|------|
| Finance | In progress | [projects/wamy-ngo/finance-module/](projects/wamy-ngo/finance-module/) |

**Context files:**
- [projects/wamy-ngo/CLAUDE.md](projects/wamy-ngo/CLAUDE.md) — project-level context
- [projects/wamy-ngo/finance-module/CLAUDE.md](projects/wamy-ngo/finance-module/CLAUDE.md) — finance module context

**Documentation:**
- [projects/wamy-ngo/finance-module/README.md](projects/wamy-ngo/finance-module/README.md) — requirements, status, folder guide

---

#### WAMY NGO — Finance Module

Double-entry accounting engine with 10 financial reports and Spring Security JWT. See [projects/wamy-ngo/finance-module/README.md](projects/wamy-ngo/finance-module/README.md) for the full requirements summary.

---

### GB Company

> Placeholder — context file stubbed, implementation pending.

**Context files:**
- [projects/gb-company/CLAUDE.md](projects/gb-company/CLAUDE.md) — placeholder context

---

## Using CLAUDE.md Files

A `CLAUDE.md` is plain Markdown that Claude Code reads as project instructions. Key sections to include:

1. **Project Overview** — one paragraph explaining what the system does and who uses it.
2. **Tech Stack** — table of technologies and versions.
3. **Layer / Folder Structure** — annotated directory tree.
4. **Naming Conventions** — table mapping artifact type to pattern and example.
5. **Critical Anti-Patterns** — things Claude must never do in this codebase.
6. **New Feature Workflow** — step-by-step checklist Claude follows when adding a feature.
7. **Open Issues** — known problems, prioritised, with file references.

The more precise and opinionated the `CLAUDE.md`, the less you need to repeat yourself in chat.

---

## Contributing

1. Add a new `projects/{project-name}/` folder.
2. Write a `CLAUDE.md` following the structure above.
3. Add a `README.md` if the project has human-facing documentation.
4. Add a row to the [Projects](#projects) table in this file.
5. Open a PR — the `CLAUDE.md` in this repo's root will guide Claude Code during review.
