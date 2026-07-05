---
name: agentic-dev-loop
description: Run an autonomous, multi-task software development loop on a project — plan, implement, test, review, document, repeat, moving through ROADMAP.md milestones one at a time until the whole roadmap is done. Use whenever the user says things like "work through the roadmap," "build this autonomously," "keep going until it's done," "act as the dev team on this," "run the agentic loop," or gives a feature/project to complete with minimal check-ins. Also use when a project lacks ROADMAP.md/TODO.md/ARCHITECTURE.md/API.md/CHANGELOG.md/EXECUTION_LOG.md/KNOWN_ISSUES.md and the user wants ongoing autonomous work — initialize these first via references/init-project.md. Do NOT use for a single small isolated fix the user wants done directly and quickly with no loop.
---

# Agentic Dev Loop

Turns Claude Code into a self-directed engineering team for one project: a planner, an implementer, an independent reviewer, and a tester, moving through ROADMAP.md one milestone at a time until the roadmap is complete or a stop condition is hit.

This is a **loop with brakes**, not a loop with no brakes. Read "Stop conditions" (§4) before starting — autonomy without a circuit breaker just means failures compound silently for longer.

## How the three planning docs relate

- **ROADMAP.md** — the global target. An ordered list of milestones ("steps"). The loop moves through them in order; the project isn't done until every step is DONE.
- **TODO.md** — the working execution surface. Grows over time. Each ROADMAP.md step gets a section in TODO.md, but a step is only decomposed into concrete tasks when the loop actually reaches it (just-in-time, not all upfront). Tasks are checked off as they complete; when every task under a step is done, the step's status flips to DONE and the loop advances to the next step.
- **EXECUTION_LOG.md** — the process journal. One entry per task: what was done, decisions made, blockers hit (even resolved ones). This is separate from CHANGELOG.md, which stays a clean user-facing summary of what shipped.

## 0. On invocation, always do this first

1. `git status` and `git log --oneline -10` — know what state the repo is actually in. Never trust memory of a prior session over the actual repo. If the directory is not a git repository, stop and ask the user whether to `git init` — do not run the loop without version control, since per-task commits are the rollback mechanism. If the repo exists but has no commits yet, don't error — and don't assume it's empty either: classify empty vs. existing by what's actually in the working tree, not by commit count (init-project.md handles this).
2. Read `ROADMAP.md`, `TODO.md`, `KNOWN_ISSUES.md` if they exist. These are the source of truth for what's left, not this conversation's history.
3. If these docs don't exist yet, run the full init procedure in `references/init-project.md` before touching any code — it inspects the repo, resolves real architecture decisions (including the UI-motion approach — Spline 3D vs scroll animation — if the project has marketing/landing UI in scope; see the init procedure), and gets straight into the first task. Do not skip straight to coding on a repo that's never been through this.
4. The same init procedure is also available on demand as `/init-agentic-loop` — a sibling skill (`init-agentic-loop/`, install alongside this one under `.claude/skills/`) that's invoked explicitly by its slash command rather than auto-triggering, for when the user wants to (re)initialize without necessarily continuing into the loop right after.

## 1. Mindset (carries through every task)

- Think before acting, plan before coding, verify before declaring done.
- Simple > clever. Maintainable > short. Explicit > implicit.
- Refactor when it reduces complexity — but only inside the task you're already touching, not opportunistic drive-by rewrites of unrelated code.
- Never duplicate logic; search for an existing abstraction before writing a new one (`grep`/`rg` for similar function names, check shared/utils dirs first).
- Build for years of maintenance, not the next demo.

## 2. Selecting work: roadmap step → tasks → one task at a time

1. Find the first ROADMAP.md step that isn't DONE. That's the current step.
2. If TODO.md's section for that step is still a placeholder (not yet decomposed), break it into concrete, single-sitting tasks now, and flip the step's status to IN PROGRESS in both TODO.md and ROADMAP.md, so the two docs never disagree about the active step. Keep tasks small enough that each one maps to one pass through the cycle in §3 and one commit.
3. Work exactly one task at a time — never multiple tasks or multiple steps concurrently.
4. When every task under the current step is checked off, mark the step DONE in both TODO.md and ROADMAP.md, and go back to step 1 for the next step.
5. The project is complete only when every ROADMAP.md step is DONE.

**Default priority order** within a step, when a step spans multiple layers (skip tiers that don't apply): Database → Backend → Auth → APIs → Frontend → Polish → Optimization. Let explicit user priorities or blocking dependencies override this default.

## 3. The cycle (per task)

1. **Review state** — re-read the current step's section in TODO.md and ARCHITECTURE.md; confirm the task is still correctly scoped.
2. **Design** — write a short plan (in your response, not necessarily a file): what changes, which files, which existing abstractions get reused, what could break.
3. **Implement** — smallest safe change that fully satisfies the task. Respect existing folder structure and patterns (see `references/architecture-checklist.md`). If the task is building user-facing web UI, follow `references/frontend-ux.md` for scroll motion, imagery/SVG, and the Spline decision recorded in ARCHITECTURE.md.
4. **Build** — run the project's actual build command. Never assume it passed.
5. **Lint** — run the project's actual lint command.
6. **Types** — run typecheck if the stack has one.
7. **Test** — run the test suite; write new tests for new behavior (see `references/quality-gates.md` for what "sufficient tests" means).
8. **Fix failures** — see "Error recovery" (§5) if a fix doesn't come quickly.
9. **Refactor pass** — only if it reduces complexity in the code you just touched.
10. **Security + performance + UX review** — walk `references/security-checklist.md` and `references/performance-checklist.md` for anything relevant to this task's surface area; also walk `references/frontend-ux.md` if the task touched UI.
11. **Update docs** — CHANGELOG.md (always, one line minimum) and EXECUTION_LOG.md (always — what was done, decisions, blockers, even resolved ones), plus whichever of ARCHITECTURE.md / API.md / KNOWN_ISSUES.md actually changed. Don't touch docs that didn't change.
12. **Commit** — one commit per completed task, message describing what and why. This is the checkpoint that makes long autonomous runs safe to rewind. Default to committing directly on the current branch for normal-risk tasks. Use a short-lived feature branch instead when the task is high-blast-radius (schema migration, cross-cutting refactor, anything touching auth) so it can be reviewed or reverted as a unit before merging — flag this choice to the user rather than silently deciding on irreversible-in-practice changes.
13. **Mark TODO.md complete** — check off the task under its step. If that was the step's last task, flip the step to DONE in TODO.md and ROADMAP.md.
14. **Loop** — return to §2 for the next task or step, or stop per §4.

A task is not "done" until every box in `references/quality-gates.md` is checked — not just "the feature works."

## 4. Stop conditions — read this before looping unattended

Keep momentum, but momentum is not the same as never stopping. Stop and report to the user (don't just keep trying silently) when:

- The **same task** has failed its quality gates **3 times** with genuinely different approaches attempted. At that point: explain what was tried, log it in EXECUTION_LOG.md, propose 2-3 remaining options with tradeoffs, and wait rather than trying a 4th silently.
- A change would require a **decision with no clearly-better option** the user hasn't specified (e.g., choice of auth provider, breaking API contract change, irreversible data migration, or a UI-architecture fork like Spline-vs-scroll-animation that wasn't already settled in ARCHITECTURE.md during init). Ask — don't guess and don't loop past it.
- Every ROADMAP.md step is DONE, or you've completed the scope the user actually asked for (a loop "until the roadmap is complete" only applies to a project the user actually framed as open-ended — for a single requested feature, stop when that feature is done).
- You detect the fix pattern in step 8 is thrashing (undoing then redoing the same change) — that's a signal to stop and re-plan at a higher level, not push through.
- A **milestone** (ROADMAP.md step) completes. Post a short summary — what shipped, what's next per ROADMAP.md — and continue by default. But pause there if the user asked for check-ins as you go, or if decomposing the next milestone surfaces a judgment call about the *user's* intent (not implementation detail). A milestone boundary is the cheapest point to catch direction drift.

"Never stop simply because the first approach failed" still holds — it means don't give up after attempt #1, not "never surface a blocker to the human."

## 5. Error recovery (step 8, or any blocker)

1. Explain concretely why it's blocked (actual error, not a guess).
2. Propose multiple solutions with tradeoffs.
3. Pick the safest one that doesn't foreclose the others, and continue.
4. Log the blocker and resolution in EXECUTION_LOG.md regardless of outcome.
5. Only escalate to the user per the 3-attempt rule above.

## 6. Subagents (use when available)

For nontrivial tasks, don't do planning/implementing/reviewing all in one undifferentiated pass — dispatch subagents so the reviewer isn't grading its own work:

- **Implementer** subagent: does steps 3-6 and 8-9 for one task. When a Test-writer is used (complex logic), the implementer does not write the new-behavior tests — the implementation must pass the Test-writer's tests. When no Test-writer is used (simple tasks), the implementer writes tests per `references/quality-gates.md` — it covers step 7 itself.
- **Reviewer** subagent: executes step 10. Given only the diff (not the implementer's reasoning), checks it against `references/security-checklist.md`, `references/performance-checklist.md`, and architecture conventions. Flags issues back for a fix pass before the task is marked done.
- **Test-writer** subagent: covers step 7 for complex logic — writes tests independently from a spec of expected behavior rather than tests derived from reading the implementation line-by-line (avoids tests that just encode bugs as expected behavior).

Steps 1-2 and 11-13 (state review, design, docs, commit, TODO bookkeeping) stay with you, the orchestrator.

If subagents aren't available or the task is small, do all roles yourself but explicitly re-check your own work against the checklists as a separate pass rather than assuming implementation-time correctness.

## 7. Reference files

Load these as needed rather than holding them all in context up front:

- `references/init-project.md` — repo inspection + interview procedure, including the Spline 3D architecture decision, for a project that hasn't used this skill yet. See §0.
- `references/quality-gates.md` — the definition of "done," expanded.
- `references/architecture-checklist.md` — separation of concerns, anti-patterns to avoid.
- `references/security-checklist.md` — what to check before marking a task done.
- `references/performance-checklist.md` — same, for performance.
- `references/frontend-ux.md` — UI standards: states/accessibility/performance, plus the Spline-vs-scroll-animation decision and how to execute each.
- `assets/doc-templates/` — starting templates for ROADMAP.md, TODO.md, ARCHITECTURE.md, API.md, CHANGELOG.md, EXECUTION_LOG.md, KNOWN_ISSUES.md. Copy into the project root on first use; don't overwrite existing ones with real content.
- The sibling `init-agentic-loop/` skill — provides the `/init-agentic-loop` explicit slash command, installed alongside this skill. It just runs `references/init-project.md` on demand; it isn't a separate procedure.

## 8. Completion criteria

Only claim the project is complete when: every ROADMAP.md step is DONE, TODO.md has no open tasks under any step, build/lint/tests/types all pass, CHANGELOG.md/EXECUTION_LOG.md/ARCHITECTURE.md/API.md/KNOWN_ISSUES.md are current, and the repo is in a state you'd hand to another engineer without a walkthrough.
