# claude-lab

Claude Code resources, guides, and coaching materials: things I experiment with, things that seem to work well,... 


## What's here

```
claude-lab/
├── guides/
│   ├── claude-code-guide.md              Full professional reference (all audiences)
│   └── claude-code-developer-guide.md    Developer-focused version — no enterprise/admin content
├── templates/
│   └── CLAUDE.md.template                Ready-to-copy CLAUDE.md starter
├── cheat-sheets/                         Quick-reference cards (coming soon)
└── sessions/                             Coaching notes and worked examples (coming soon)
```

---

## Guides

### [claude-code-guide.md](guides/claude-code-guide.md)

The full reference. Covers:

- What Claude Code actually is (and isn't)
- CLAUDE.md and auto memory; how they load, how to write them effectively
- Hooks: all four types, every lifecycle event, configuration schema, exit codes
- MCP servers — configuration, permissions, security considerations
- Skills, subagents, plugins
- Settings files and permission modes
- Context management
- Enterprise deployment checklist
- A catalogue of features that don't exist but are commonly cited

### [claude-code-developer-guide.md](guides/claude-code-developer-guide.md)

The same content with the enterprise/admin sections removed. For individual contributors and small teams who don't need managed policy configuration, org-wide hooks, or compliance checklists.

---

## Templates

### [CLAUDE.md.template](templates/CLAUDE.md.template)

A ready-to-copy CLAUDE.md starter based on the principle that Claude can read your code — what it cannot read is your intent.

The template is structured around sections that matter:

- Non-obvious architecture decisions
- Known issues and active workarounds
- Decisions made and why
- Hard constraints (what not to do)
- Build and test commands
- Naming conventions
- Domain glossary

Copy it to your project root, fill in the placeholders, delete what doesn't apply. Target under 200 lines — longer files reduce adherence.

---

## How to use the template

```bash
# Copy into your project
cp path/to/claude-lab/templates/CLAUDE.md.template your-project/CLAUDE.md

# Edit it
# - Replace [bracketed placeholders] with your specifics
# - Delete sections that don't apply
# - Keep it under 200 lines

# Let Claude generate a first draft from your codebase (optional starting point)
cd your-project && claude
/init
```

The `/init` command generates a CLAUDE.md from your codebase. It's a useful starting point but tends to over-document obvious things. Use the template to structure what Claude cannot discover from code alone.

---

## What this repo is not

- Not a tutorial. The guides assume you have Claude Code installed and are configuring it.
- Not exhaustive. Features are included when verified, not because they're anticipated.
- Not a community resource. Content here is held to the same standard as official documentation.

If something looks wrong or outdated, the source of truth is code.claude.com/docs.
