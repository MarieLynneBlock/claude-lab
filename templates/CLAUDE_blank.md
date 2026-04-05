# [Project Name]

<!--
  FILE NAMING: This file is named CLAUDE_blank.md to distinguish it from other
  templates in claude-lab/templates/. When copying to a project, rename it to
  CLAUDE.md — the "_blank" suffix must be removed for Claude Code to load it.

  CLAUDE.md — Project Memory
  
  This file is loaded into Claude's context at the start of every session.
  It is context, not enforced configuration — specificity and brevity directly
  affect how consistently Claude follows it.
  
  Target: under 200 lines. If it grows beyond that, split using .claude/rules/.
  Rule of thumb: write what Claude cannot discover by reading your code.
  Delete any section that doesn't apply — a shorter, focused file outperforms
  a long one with half the fields left as placeholders.
  
  This template is maintained in claude-lab/templates/
-->

---

## What this project is

<!--
  2–4 sentences. What does this codebase do? Who uses it? What problem does it solve?
  Not a README — Claude doesn't need the marketing pitch. It needs enough context
  to understand decisions it will encounter in the code.
-->

[Description]

**Primary language(s):** [e.g. Python 3.11 / TypeScript 5 / Java 17]  
**Framework(s):** [e.g. FastAPI / Next.js / Spring Boot]  
**Deployment target:** [e.g. AWS EKS / Azure / on-prem]

---

## Build and test commands

<!--
  Be exact. Claude will run these. Vague instructions produce wrong commands.
  Include prerequisites if a command requires local services, specific env vars, etc.
-->

| Task | Command | Notes |
|---|---|---|
| Install dependencies | `[command]` | |
| Build | `[command]` | |
| Unit tests | `[command]` | |
| Integration tests | `[command]` | Requires: [e.g. local PostgreSQL on port 5433] |
| Lint | `[command]` | Must pass before any PR |
| Run locally | `[command]` | |

---

## Architecture decisions that are not obvious from the code

<!--
  This is the highest-value section. Document WHY, not WHAT.
  Claude can read what the code does. It cannot read why a decision was made,
  or why the "obvious" refactor would break something.
  
  Examples of what belongs here:
  - "We use X pattern for Y — do not replace with Z even if it looks redundant"
  - "Service A is deprecated; new code must use Service B"
  - "Authentication is handled by middleware X; never add auth logic to controllers"
  
  Link to ADRs if you have them.
-->

- [Decision and why]
- [Decision and why]

---

## What not to do

<!--
  Explicit constraints Claude must respect. Be direct.
  The more specific, the more reliably it is followed.
  
  Examples:
  - "Do not commit directly to main — all changes via PR"
  - "Do not use console.log in production code — use the logger at src/lib/logger.ts"
  - "Do not install packages without updating docs/dependencies.md"
  - "Do not refactor [X] — it has a known issue tracked in ticket #NNN"
-->

- [Constraint]
- [Constraint]

---

## Known issues and active workarounds

<!--
  Things Claude might try to "fix" that must not be touched.
  Include the ticket or ADR reference so the reasoning is traceable.
-->

- `[function/file]` — [what the issue is]. Do not refactor. [Ticket / ADR link]
- Tests in `[path]` require [specific setup] — see [doc link]

---

## Domain vocabulary

<!--
  Terms that mean something specific in this codebase that differ from
  common usage. Without this, Claude will use the common interpretation.
  
  Examples:
  - "Member" ≠ "User" — a User has an account; a Member has a linked subscription
  - "Policy" means insurance policy object, not a permission policy
  - "Event" in this codebase refers to [X], not a DOM event or calendar event
-->

| Term | What it means here |
|---|---|
| [Term] | [Definition] |

---

## Code conventions

<!--
  Only include things that differ from language/framework defaults,
  or that Claude gets wrong without explicit guidance.
  Stack-specific: delete what doesn't apply.
-->

**Naming:**

- [e.g. React components: PascalCase]
- [e.g. API handlers: handleVerbResource — handleCreateUser]
- [e.g. Database models: snake_case]

**Patterns we use:**

- [e.g. Prefer X over Y for Z reason]
- [e.g. All API responses follow the shape at src/types/api.ts]

**Patterns we avoid:**

- [e.g. No direct database access outside of the repository layer]

---

## Git and PR workflow

<!--
  Delete if your team follows standard conventions with no exceptions.
  Only document what differs or what Claude gets wrong.
-->

- Branch naming: [e.g. `feat/`, `fix/`, `chore/`]
- Commit convention: [e.g. Conventional Commits — `feat:`, `fix:`, `docs:`]
- PR requirements: [e.g. one reviewer, all tests green, lint passing]

---

## Slash commands available in this project

<!--
  Commands defined in .claude/commands/ that are relevant to this project.
  Remove this section if none are defined.
-->

| Command | What it does |
|---|---|
| `/[command]` | [Description] |

---

<!--
  Maintainer notes (stripped from Claude's context, visible to humans only):
  
  Last updated: [date]
  Updated by: [name]
  Review when: [e.g. after major refactor, quarterly]
  
  Sections intentionally left blank: [list any you removed and why]
-->
