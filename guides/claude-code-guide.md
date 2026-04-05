# Claude Code: Professional Reference Guide

> **Source**: Verified against official Claude Code documentation (code.claude.com/docs). April 2026.  
> Intended as the factual basis for a team cheat sheet — covering what actually exists, not community folklore.

---

## 1. What Claude Code Is

Claude Code is an **agentic AI development tool** that runs in your terminal (and increasingly in IDEs and CI/CD pipelines). It is not a code autocomplete engine. It can read your codebase, execute commands, manage files, orchestrate subagents, and connect to external systems. Think of it as a programmable, extensible AI agent with a deterministic configuration layer on top.

The extension stack, in order of introduction:

| Layer | What it is | Defined in |
|---|---|---|
| **MCP Servers** | External tool connections (APIs, databases, services) | `.mcp.json` |
| **CLAUDE.md** | Persistent project/user instructions Claude reads each session | `CLAUDE.md`, `.claude/rules/` |
| **Skills** | Scoped instruction packages that activate for specific tasks | `.claude/skills/` or plugin |
| **Slash Commands** | Repeatable, user-triggered workflows | `.claude/commands/*.md` |
| **Subagents** | Parallel isolated agents with their own context window | `.claude/agents/*.md` |
| **Hooks** | Shell commands, HTTP calls, or LLM prompts that fire at lifecycle events | `settings.json` |
| **Plugins** | Bundled, shareable configurations: commands + agents + skills + hooks + MCP | `.claude-plugin/plugin.json` |

---

## 2. Memory: CLAUDE.md and Auto Memory

Claude Code has two distinct memory mechanisms. Both are loaded at session start.

### 2.1 CLAUDE.md Files (You Write)

Plain markdown files that give Claude persistent context. Claude treats them as **context, not enforced configuration** — specificity and brevity directly affect adherence.

**Location hierarchy** (more specific overrides broader):

| Scope | Location | Shared? |
|---|---|---|
| Organisation-wide (managed) | `/etc/claude-code/CLAUDE.md` (Linux/WSL) | All users via IT/DevOps |
| Project | `./CLAUDE.md` or `./.claude/CLAUDE.md` | Team, via version control |
| Personal (all projects) | `~/.claude/CLAUDE.md` | Just you |
| Personal (this project) | `./CLAUDE.local.md` (add to `.gitignore`) | Just you |

**How files load**: Claude walks the directory tree upward from your working directory, loading all CLAUDE.md and CLAUDE.local.md files it finds. Subdirectory CLAUDE.md files load lazily when Claude accesses those directories.

**Writing effective instructions**:

- **Target under 200 lines per file**. Longer files consume context and reduce adherence. Split using imports or `.claude/rules/`.
- **Be concrete and verifiable**: "Run `npm test` before committing" beats "test your changes."
- **Use markdown headers and bullets** — Claude scans structure the way humans do.
- **No contradictions**: conflicting rules across nested CLAUDE.md files produce arbitrary behaviour.
- **Use `@path/to/file` syntax** to import additional files (READMEs, package.json, docs). Max import depth: 5 hops.
- **HTML block comments** (`<!-- maintainer notes -->`) are stripped from context but visible to humans editing the file. Use them for notes that don't need to burn tokens.

**Cross-tool compatibility**: If your repo already has `AGENTS.md` for other agents, create a `CLAUDE.md` that imports it:

```markdown
@AGENTS.md

## Claude Code
Use plan mode for changes under `src/billing/`.
```

**Scoped rules** (`.claude/rules/*.md`): For large projects, break instructions into topic-specific files with optional `paths:` frontmatter to scope rules to specific file types or directories. Rule files load on demand when Claude accesses matching paths.

**Run `/init`** to generate a starting CLAUDE.md from your codebase. Set `CLAUDE_CODE_NEW_INIT=1` for an interactive multi-phase flow.

---

### 2.2 Auto Memory (Claude Writes)

Claude can write its own notes based on corrections and preferences you give during sessions. These load at session start (first 200 lines or 25KB).

- Stored per working tree
- Enable/disable via settings or `/memory` command
- View and edit with `/memory`
- Subagents can maintain their own auto memory separately

**Use CLAUDE.md for** deliberate standards, architecture, workflows.  
**Let auto memory handle** build commands, debugging insights, preferences Claude discovers from your corrections.

---

## 3. Hooks: Deterministic Lifecycle Automation

Hooks are the most powerful and most misrepresented Claude Code feature. They are **deterministic** — they run every time, unlike CLAUDE.md instructions which Claude may follow inconsistently. Use them for rules that must not be optional.

### 3.1 Four Hook Types

| Type | What it runs | Use case |
|---|---|---|
| `command` | Shell script (stdin = JSON event) | Blocking dangerous commands, running linters, sending alerts |
| `http` | HTTP POST to an endpoint (body = JSON event) | Centralised validation services, audit logging |
| `prompt` | Single-turn LLM evaluation | Nuanced content checks that need reasoning |
| `agent` | Subagent with tool access (Read, Grep, Glob) | Contextual verification that requires codebase inspection |

### 3.2 Hook Locations and Scope

| Location | Scope | Committed to repo? |
|---|---|---|
| `~/.claude/settings.json` | All your projects | No |
| `.claude/settings.json` | This project | Yes |
| `.claude/settings.local.json` | This project | No (gitignored) |
| Managed policy settings | Organisation-wide | Admin-controlled |
| Plugin `hooks/hooks.json` | When plugin is enabled | Yes, bundled with plugin |
| Skill or agent frontmatter | While that component is active | Yes |

Enterprise: `allowManagedHooksOnly` in managed settings blocks user/project/plugin hooks — only admin-defined hooks run.

### 3.3 Complete Hook Event Reference

Events are listed in lifecycle order:

**Session lifecycle**

| Event | When it fires | Can block? |
|---|---|---|
| `SessionStart` | Session begins or resumes | No (but adds context) |
| `InstructionsLoaded` | A CLAUDE.md or rules file loads | No (observability only) |
| `UserPromptSubmit` | User submits a prompt, before Claude processes it | Yes (blocks and erases prompt) |
| `PreCompact` | Before compaction runs | No |
| `PostCompact` | After compaction completes | No |
| `SessionEnd` | Session terminates | No |

**Agentic loop**

| Event | When it fires | Can block? |
|---|---|---|
| `PreToolUse` | Before a tool call executes | Yes (deny/allow/ask/defer) |
| `PermissionRequest` | When a permission dialog would appear | Yes (allow or deny on user's behalf) |
| `PermissionDenied` | When auto mode classifier denies a tool call | No (can signal retry) |
| `PostToolUse` | After a tool call succeeds | No (sends feedback to Claude) |
| `PostToolUseFailure` | After a tool call fails | No (sends feedback to Claude) |
| `Stop` | When Claude finishes responding | Yes (can force Claude to continue) |
| `StopFailure` | When a turn ends due to API error | No (logging only) |

**Subagents and teams**

| Event | When it fires | Can block? |
|---|---|---|
| `SubagentStart` | When a subagent is spawned | No (adds context to subagent) |
| `SubagentStop` | When a subagent finishes | Yes |
| `TaskCreated` | When a task is created via TaskCreate tool | Yes (rolls back creation) |
| `TaskCompleted` | When a task is marked completed | Yes (prevents completion) |
| `TeammateIdle` | When an agent team teammate is about to go idle | Yes (forces teammate to continue) |

**Environment and configuration**

| Event | When it fires | Can block? |
|---|---|---|
| `Notification` | When Claude Code sends a notification | No |
| `ConfigChange` | When a config file changes during session | Yes (except `policy_settings`) |
| `CwdChanged` | When working directory changes | No |
| `FileChanged` | When a watched file changes | No |
| `WorktreeCreate` | When a worktree is created | Yes (replaces default git behaviour) |
| `WorktreeRemove` | When a worktree is removed | No |
| `Elicitation` | When MCP server requests user input | Yes (respond programmatically) |
| `ElicitationResult` | After user responds to MCP elicitation | Yes (override response) |

### 3.4 Hook Configuration Schema

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "if": "Bash(rm *)",
            "command": "\"$CLAUDE_PROJECT_DIR\"/.claude/hooks/block-rm.sh",
            "timeout": 30
          }
        ]
      }
    ]
  }
}
```

**Exit codes for command hooks**:

| Exit code | Meaning |
|---|---|
| `0` | Success. Claude Code parses stdout for JSON output. |
| `2` | Blocking error. stderr is fed to Claude as an error message. Exact effect depends on event (see table above). |
| Anything else | Non-blocking error. Execution continues. |

**Key JSON output fields** (stdout on exit 0):

```json
{
  "continue": false,
  "stopReason": "Message shown to user (not Claude)",
  "decision": "block",
  "reason": "Shown to Claude when blocking",
  "hookSpecificOutput": {
    "hookEventName": "PreToolUse",
    "permissionDecision": "deny",
    "permissionDecisionReason": "Shown to Claude"
  }
}
```

**PreToolUse `permissionDecision` values**: `allow` | `deny` | `ask` | `defer`  
When multiple PreToolUse hooks return different decisions: `deny` > `defer` > `ask` > `allow`

### 3.5 Matcher Patterns

The `matcher` field is a **regex** string. Examples:

```
"Bash"              # exact tool name
"Edit|Write"        # either tool
"mcp__github__.*"   # all tools from the github MCP server
"mcp__.*__write.*"  # any MCP tool with "write" in the name
```

MCP tools follow the pattern `mcp__<server>__<tool>`. Example: `mcp__github__list_issues`.

For tool events, add an `if` field on individual handlers for finer matching:  
`"Bash(git *)"` → only git commands  
`"Edit(*.ts)"` → only TypeScript files

### 3.6 Environment Variables in Hooks

`SessionStart`, `CwdChanged`, and `FileChanged` hooks have access to `$CLAUDE_ENV_FILE`. Variables written here persist into all subsequent Bash commands for the session:

```bash
#!/bin/bash
if [ -n "$CLAUDE_ENV_FILE" ]; then
  echo 'export NODE_ENV=production' >> "$CLAUDE_ENV_FILE"
fi
```

**Reference scripts by path safely**:
- `$CLAUDE_PROJECT_DIR` — project root (quote to handle spaces)
- `${CLAUDE_PLUGIN_ROOT}` — plugin installation directory
- `${CLAUDE_PLUGIN_DATA}` — plugin persistent data directory

---

## 4. MCP Servers: External Connections

MCP (Model Context Protocol) is an open standard for connecting Claude to external tools and data sources. MCP servers can run as local processes or over HTTP.

### 4.1 Configuration (`.mcp.json`)

```json
{
  "mcpServers": {
    "github": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@anthropic/mcp-github"],
      "env": {
        "GITHUB_TOKEN": "${GITHUB_TOKEN}"
      }
    },
    "postgres": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@anthropic/mcp-postgres"],
      "env": {
        "DATABASE_URL": "${DATABASE_URL}"
      }
    }
  }
}
```

**Always use environment variable interpolation for secrets** — never hardcode tokens in `.mcp.json`.

### 4.2 MCP Tool Permissions

MCP tools require **explicit permission** before Claude can use them. Use `allowedTools` with wildcard matching:

```json
{
  "allowedTools": [
    "mcp__github__*",
    "mcp__db__query",
    "mcp__slack__send_message"
  ]
}
```

`permissionMode: "acceptEdits"` does **not** auto-approve MCP tools — it only covers file edits and filesystem Bash commands. Use `allowedTools` instead.

### 4.3 Security Considerations for MCP

Real CVEs have been filed against MCP server implementations (CVE-2025-68143, CVE-2025-68144, CVE-2025-68145 against `mcp-server-git`). Lessons that generalise:

- **Path validation is not optional**: servers restricted to a repo path must validate that tool calls stay within it (symlink-resolved).
- **Principle of least privilege**: `allowedTools` should enumerate only what's needed, not `mcp__<server>__*` blanket access for sensitive servers.
- **Familiar tools are not automatically safe wrappers**: Git/filesystem tools exposed via MCP inherit the attack surface of the underlying system.
- **Scope MCP configs appropriately**: `.mcp.json` at project scope is shareable; move sensitive auth to `.claude/settings.local.json` or environment variables.

### 4.4 Well-Known MCP Servers

| Server | Purpose |
|---|---|
| GitHub | PRs, issues, repos |
| JIRA/Linear | Ticket workflows |
| Slack | Notifications, search |
| PostgreSQL | Direct database queries |
| Playwright | Browser automation |
| Filesystem | Scoped file access |

---

## 5. Skills: Reusable Instruction Packages

Skills are markdown files with frontmatter that give Claude specialised knowledge or behaviour for specific task types. They activate on task match (not automatically — Claude or the user triggers them).

### 5.1 Structure

```
.claude/skills/
└── code-review/
    ├── SKILL.md        # Instructions and metadata
    ├── scripts/        # Executable automation
    ├── references/     # Docs loaded on demand
    └── assets/         # Templates and static files
```

### 5.2 SKILL.md frontmatter

```yaml
---
name: secure-operations
description: Perform operations with security checks
hooks:
  PreToolUse:
    - matcher: "Bash"
      hooks:
        - type: command
          command: "./scripts/security-check.sh"
---
```

Skills can define their own hooks, scoped to when the skill is active. Use `once: true` on a hook handler to run it only once per session.

---

## 6. Subagents: Parallel Isolated Work

Subagents are specialised Claude instances that run in their **own context window**, preventing context pollution in the main session. They return a summary rather than dragging all their intermediate work back up.

### 6.1 Agent Definition (`.claude/agents/`)

```yaml
---
name: security-auditor
description: Specialised agent for security analysis and vulnerability assessment
model: claude-opus-4-5
---

You are a security expert. Your responsibilities:
- Identify injection vulnerabilities
- Check for hardcoded secrets
- Review authentication patterns
- Assess input validation

Always provide: severity rating, specific line references, remediation steps.
```

### 6.2 Spawning Subagents

Subagents are spawned via the **Agent tool** during a session, not via static configuration at startup. You can also use `claude --agent <name>` from the CLI to start a session with a specific agent active.

**Isolation modes**:
- Default: subagent runs in the main working directory
- `isolation: "worktree"` — subagent gets its own git worktree (useful for parallel branches)

### 6.3 Context Budget Awareness

"Claude drifted" complaints are often **context budget complaints**. Anthropic's docs note that as the context fills — conversation history, file reads, CLAUDE.md, auto memory, loaded skills, system instructions — Claude compacts automatically. Older tool outputs are cleared first, then conversation is summarised. Use subagents proactively for heavy computational tasks to keep the main context clean.

---

## 7. Plugins: Shareable Configuration Bundles

Plugins package multiple Claude Code components for distribution across teams or projects.

### 7.1 Plugin Structure

```
plugin-name/
├── .claude-plugin/
│   └── plugin.json     # Plugin metadata and manifest
├── commands/           # Slash commands
├── agents/             # Specialised agents
├── skills/             # Agent skills
├── hooks/
│   └── hooks.json      # Hook definitions
├── .mcp.json           # MCP server configuration
└── README.md
```

Plugins are the right unit of distribution when you want consistent tooling enforced across a team without every developer maintaining their own `.claude/` directory.

---

## 8. Settings Files: Permissions and Configuration

### 8.1 Settings File Hierarchy

```
~/.claude/settings.json          # User (all projects)
.claude/settings.json            # Project (shareable)
.claude/settings.local.json      # Project (personal, gitignored)
Managed policy settings          # Organisation (admin-controlled)
```

### 8.2 Key settings.json Structure

```json
{
  "permissions": {
    "allow": ["Bash(npm run *)", "Edit(src/*)"],
    "deny": ["Bash(rm -rf *)"]
  },
  "hooks": {
    "PreToolUse": [...],
    "PostToolUse": [...],
    "Stop": [...]
  },
  "env": {
    "MAX_THINKING_TOKENS": "10000",
    "CLAUDE_AUTOCOMPACT_PCT_OVERRIDE": "50"
  }
}
```

### 8.3 Permission Modes

| Mode | Behaviour |
|---|---|
| `default` | Standard permission prompts |
| `plan` | Claude plans before acting; requires approval to execute |
| `acceptEdits` | Auto-approves file edits; still prompts for Bash |
| `auto` | Auto mode — classifier decides; fires `PermissionDenied` hooks when blocked |
| `dontAsk` | Skips most prompts |
| `bypassPermissions` | No permission checks (use only in sandboxed environments) |

---

## 9. Context Management

| Context level | Recommended action |
|---|---|
| 0–50% | Normal operation |
| 50–70% | Monitor; consider spawning subagents for heavy tasks |
| 70–80% | Run `/compact` (summarises, retains instructions) |
| 80%+ | `/compact` mandatory; or `/clear` to start fresh |

**`/compact` vs `/clear`**:  
`/compact` summarises older context but loses fidelity on details — instructions from CLAUDE.md are re-loaded but conversation specifics are condensed.  
`/clear` wipes everything and starts a fresh session from CLAUDE.md only.  
Use `/compact` when mid-task continuity matters; `/clear` when switching to an unrelated task.

**`PreCompact` and `PostCompact` hooks** let you react to compaction events — e.g., logging the generated summary, updating external state, or injecting additional context post-compaction.

---

## 10. A CLAUDE.md Template That Actually Works

The goal is not to list your tech stack (Claude can read your files). The goal is to communicate what Claude **cannot** discover from the code itself.

```markdown
# Project: [Name]

## Non-obvious architecture decisions
- Authentication is handled by middleware X; never add auth logic to controllers directly.
- We use Y pattern for Z — do not replace with the "obvious" alternative even if it looks redundant.
- Service A is deprecated; new code must use Service B.

## Known issues and active workarounds
- `foo()` in `src/utils/legacy.ts` has a race condition under load. Do not refactor it — ticket #1234 is open.
- Tests in `tests/integration/payments/` require a running local PostgreSQL. See `docker-compose.local.yml`.

## Decisions made and why
- We chose Library X over Y because of Z. If you are tempted to switch, read `docs/adr/002-library-choice.md` first.

## What not to do
- Do not commit directly to `main`. All changes via PR.
- Do not add `console.log` to production code; use the structured logger at `src/lib/logger.ts`.
- Do not install new npm packages without updating `docs/dependencies.md`.

## Build and test
- `npm run build` — full build
- `npm test` — unit tests only
- `npm run test:integration` — requires local services (see above)
- `npm run lint` — must pass before PR

## Naming conventions
- React components: PascalCase
- API handlers: `handle[Verb][Resource]` (e.g., `handleCreateUser`)
- Database models: snake_case

## Domain glossary (terms that mean something specific here)
- "Member" ≠ "User". A User has an account; a Member has a subscription linked to a policy.
- "Policy" refers to the insurance policy object, not a permission policy.
```

---

## 11. Getting Started (Verified Steps)

```bash
# Install
npm install -g @anthropic-ai/claude-code

# Navigate to your project
cd my-project && claude

# Generate an initial CLAUDE.md from your codebase
/init

# View your configured hooks
/hooks

# View and edit auto memory
/memory

# Add an MCP server
claude mcp add <server-name> -- npx -y @anthropic/<server-package>

# Run in non-interactive/headless mode
claude -p "Your prompt here"

# Resume a previous session
claude --resume <session-id>

# Start with a specific agent
claude --agent security-auditor
```

---

## 12. Enterprise Checklist

For teams deploying Claude Code at scale:

- [ ] **Managed CLAUDE.md** at `/etc/claude-code/CLAUDE.md` for org-wide coding standards
- [ ] **`allowManagedHooksOnly`** in policy settings to prevent users bypassing security hooks
- [ ] **PreToolUse hooks** for blocking prohibited patterns (secrets in code, destructive commands)
- [ ] **PostToolUse hooks** for automatic lint/test feedback after file writes
- [ ] **MCP auth via environment variables** — never tokens in `.mcp.json` committed to repos
- [ ] **Scoped `allowedTools`** for MCP — least privilege, not wildcard access for sensitive servers
- [ ] **`.claude/settings.json` in version control** for team-shared config; `.claude/settings.local.json` gitignored for personal overrides
- [ ] **Cost monitoring** — Claude Code API calls in agentic sessions can accumulate quickly. Track usage via Claude Code Analytics API
- [ ] **Audit hooks** using `InstructionsLoaded` and `ConfigChange` events for compliance tracing
- [ ] **Plugin strategy** for standardised team tooling rather than every developer maintaining their own `.claude/` directory

---

## 13. What Doesn't Exist (Common Misinformation)

For completeness, features often cited online that are **not real**:

| Claimed feature | Reality |
|---|---|
| `PostToolLive` hook | Does not exist. The hook is `PostToolUse`. |
| `SessionSmt` hook | Does not exist. Possibly a garbled `SessionStart`. |
| Auto-activating skills by name | Skills activate via task match or explicit invocation, not automatic name detection. |
| YAML agent definition files with `.yml` extension | Agent definitions are markdown files with YAML frontmatter, stored in `.claude/agents/`. |
| SKILL.md as a built-in Claude Code feature | Skills are real, but the SKILL.md convention and auto-activation logic is primarily a community pattern built on top of the core feature — not a native spec. |

---

*Maintained against: code.claude.com/docs — always verify against current docs before implementation.*
