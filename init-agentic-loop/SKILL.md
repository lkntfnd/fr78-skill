---
name: init-agentic-loop
description: Explicitly bootstrap the agentic-dev-loop project docs for this repo. Invoke directly as /init-agentic-loop.
disable-model-invocation: true
---

# Init Agentic Loop

This skill exists only to be invoked explicitly as `/init-agentic-loop` — it does not auto-trigger from conversation, since the `agentic-dev-loop` skill already detects missing docs and runs this same procedure on its own when needed. Use this when you want to (re)initialize a project's docs on demand without necessarily continuing straight into the loop.

Read and follow `../agentic-dev-loop/references/init-project.md` in full — repo inspection, the interview, then synthesizing ROADMAP.md, TODO.md, ARCHITECTURE.md, API.md, CHANGELOG.md, EXECUTION_LOG.md, and KNOWN_ISSUES.md from `../agentic-dev-loop/assets/doc-templates/`.

This assumes `agentic-dev-loop` is installed as a sibling skill directory (both under the same `.claude/skills/` or `~/.claude/skills/`). If that relative path doesn't resolve, check both standard locations — project-level `.claude/skills/agentic-dev-loop/references/init-project.md` and user-level `~/.claude/skills/agentic-dev-loop/references/init-project.md` — and use whichever exists (the two skills may be installed at different scopes). Only if it's found in neither, tell the user the `agentic-dev-loop` skill needs to be installed alongside this one.

On a project whose roadmap is already fully DONE, this command instead runs init-project.md's **extension mode**: review the finished project, derive the next phase's milestones, present them — and stop.

Stop after presenting the summary per init-project.md Step 4 — don't continue into the main loop's task execution unless the user says to (this command is for initializing/re-initializing on demand, not for running the loop itself).
