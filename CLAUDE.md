# CLAUDE.md — claude-code-for-developers

**Purpose:** This repo teaches developers how to use Claude Code effectively on real software projects.  
**Audience:** Software engineers learning AI-assisted development workflows.  
**Maintainer:** Tahajjat Tuhin — tahajjat.tuhin@gmail.com

---

## 1. Repo Purpose

This is a **reference and teaching repository**, not a runnable application. Its value is in the context files (`CLAUDE.md`, `README.md`) and the project structures they describe. When Claude Code reads this repo, it should:

- Understand that each `projects/` subfolder represents a distinct production or representative codebase.
- Respect the per-project and per-module `CLAUDE.md` files as the authoritative source of truth for that scope.
- Never assume conventions from one project apply to another.

---

## 2. Repo Structure

```
claude-code-for-developers/
├── CLAUDE.md                          ← You are here (root context)
├── README.md                          ← Human-facing overview with TOC
└── projects/
    ├── wamy-ngo/
    │   ├── CLAUDE.md                  ← WAMY NGO project context
    │   └── finance-module/
    │       ├── CLAUDE.md              ← Finance module context
    │       └── README.md              ← Finance module docs
    └── gb-company/
        └── CLAUDE.md                  ← GB Company placeholder context
```

---

## 3. How Context Files Are Layered

Claude Code loads `CLAUDE.md` files from the current directory upward. This repo uses three levels:

| Level | File | Scope |
|-------|------|-------|
| Root | `CLAUDE.md` | Repo-wide rules (this file) |
| Project | `projects/{name}/CLAUDE.md` | Stack, architecture, conventions for one project |
| Module | `projects/{name}/{module}/CLAUDE.md` | Narrowed context for one bounded domain |

**Rule:** a lower-level file adds to — and may override — a higher-level file for the same scope. It never replaces repo-wide rules.

---

## 4. Writing Guidelines for Context Files

When adding or editing a `CLAUDE.md` in this repo, follow these rules:

- **Be opinionated.** Vague guidance produces vague output. State rules as imperatives.
- **Include anti-patterns.** Telling Claude what NOT to do is as important as what to do.
- **Version your stack.** Write `Spring Boot 3.2`, not `Spring Boot`. Versions change behaviour.
- **Provide canonical examples.** A short code snippet beats three paragraphs of prose.
- **Link open issues.** Note known problems with file paths so Claude can find them.
- **Keep it current.** A stale `CLAUDE.md` is worse than none — it will mislead Claude.

---

## 5. AI Collaboration Rules (Repo-Wide)

- Never generate runnable application code in this repo unless a project subfolder explicitly contains source files.
- When editing a `CLAUDE.md`, preserve existing section numbers and headings unless asked to restructure.
- When adding a new project, create the folder and `CLAUDE.md` before any other files.
- Always update `README.md`'s Projects table when a new project folder is added.
- Do not invent conventions for a project — ask if the `CLAUDE.md` for that project does not cover the case.

---

## 6. Current Projects

| Project | Stack | CLAUDE.md |
|---------|-------|-----------|
| WAMY NGO ERP | Spring Boot 3.2, React 19, MySQL 8 | [projects/wamy-ngo/CLAUDE.md](projects/wamy-ngo/CLAUDE.md) |
| GB Company | TBD | [projects/gb-company/CLAUDE.md](projects/gb-company/CLAUDE.md) |
