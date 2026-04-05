# [Project Name]

<!--
  FILE NAMING: This file is named CLAUDE.local_blank.md. When copying to a project, rename it to CLAUDE.local.md — the "_blank" suffix must be removed.

  CLAUDE.local.md: Personal Project Memory
  
  This file is your private overlay on top of the shared CLAUDE.md.
  It is loaded alongside CLAUDE.md at the start of every session.
  It is NOT committed to version control — add *.local.md to .gitignore.

  Use it for:
  - Your role on the project and what Claude should assume about your knowledge
  - How you want Claude to communicate with you (tone, length, format)
  - Personal preferences that don't belong in the shared project file
  - Local environment details (ports, paths, credentials locations)
  - Workflow habits you want Claude to follow for your sessions only
  - Constraints or context that are specific to your machine or role

  Keep it short. If something is true for the whole team, it belongs in CLAUDE.md instead.

  This template is maintained in claude-lab/templates/
-->

---

## My role on this project

<!--
  Who you are in relation to this codebase. Claude uses this to calibrate
  what to explain versus assume, and how to frame suggestions.

  Examples:
  - "I'm the backend lead. I don't own the frontend — explain React patterns, don't assume I know them."
  - "I'm a new joiner. Err on the side of explaining context."
  - "I'm the sole engineer. No need to flag cross-team impact."
-->

[Your role and what Claude should assume about your knowledge]

---

## How to work with me

<!--
  Personal communication preferences for your Claude sessions.
  These cannot go in the shared CLAUDE.md — they are about you, not the project.

  Examples:
  - "Keep responses short. I can read the diff."
  - "Don't summarize what you just did at the end of a response."
  - "When I ask for options, give me 2–3 with a recommendation — don't just list tradeoffs."
  - "Explain backend concepts using Go analogues, not JavaScript."
  - "Flag security issues immediately, even if I didn't ask."
-->

- [Preference]
- [Preference]

---

## Non-obvious decisions

<!--
  Decisions you've made locally that override or extend the shared CLAUDE.md.
  Include the reason — otherwise future-you won't know if it still applies.
-->

- [Decision and why]

---

## What not to do

<!--
  Personal constraints for your sessions. Things Claude gets wrong for you
  specifically, or shortcuts you want blocked.
-->

- [Constraint]

---

## Local environment

<!--
  Machine-specific details Claude needs to run commands correctly.
  Delete anything that matches the project defaults in CLAUDE.md.
-->

- Local DB port: [e.g. 5433 — non-standard because 5432 is taken by another project]
- Credentials: [e.g. stored in ~/.config/project/.env — never commit]
- [Any other machine-specific path or service detail]

---

## Git workflow

<!--
  Only include if your personal workflow differs from the shared convention in CLAUDE.md.
  Common reason: you squash locally before pushing, or use a different branch prefix.
-->

- [Personal deviation from team convention, and why]

---

<!--
  Maintainer notes (not loaded by Claude):

  Last updated: [date]
  Keep in sync with: CLAUDE.md (shared project file)
  Review when: local environment changes, or after onboarding to a new machine
-->
