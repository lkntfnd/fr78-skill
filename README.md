# Agentic Dev Loop

**A Claude Code skill that turns Claude into a self-directed engineering team — planning, implementing, testing, reviewing, and documenting its way through a roadmap, milestone by milestone, with built-in brakes.**

![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg) ![Claude Code](https://img.shields.io/badge/Claude%20Code-Skill-d97757.svg) ![Docs as State](https://img.shields.io/badge/state-docs%20%2B%20git-success.svg)

The loop is stateless by design: all project state lives in seven markdown docs and git history, so any session — including one that lost its context — resumes exactly where the last one stopped.

## How it works

```mermaid
flowchart TD
    A([Invocation]) --> B{Git repo?}
    B -- no --> B1[Stop — offer git init]
    B -- yes --> C{Project docs exist?}
    C -- no --> D["Init: inspect repo → interview →<br/>7 docs + reviewed roadmap → docs-only commit"]
    C -- yes --> E[Read ROADMAP · TODO · KNOWN_ISSUES]
    D --> E
    E --> F[Pick first non-DONE milestone]
    F --> G[Decompose into tasks<br/>just-in-time]
    G --> H[[Per-task cycle]]
    H --> I{Milestone<br/>complete?}
    I -- no --> H
    I -- yes --> J[Post milestone summary]
    J --> K{Roadmap<br/>complete?}
    K -- no --> F
    K -- yes --> L([Done — handoff-ready repo])
```

Every task runs the same gated cycle — one task, one commit, one rewind point:

```mermaid
flowchart LR
    S1[Design] --> S2[Implement] --> S3[Build · Lint ·<br/>Types · Test]
    S3 --> S4{Quality<br/>gates?}
    S4 -- "fail (&lt;3 attempts)" --> S2
    S4 -- "3 failures" --> STOP([Stop — report options to user])
    S4 -- pass --> S5[Security · Perf ·<br/>UX review] --> S6[Update docs] --> S7[Commit] --> S8[Check off<br/>in TODO.md]
```

## The seven project docs

| Doc | Role |
|---|---|
| `ROADMAP.md` | Global target — 2–5 outcome-based milestones with verifiable exit criteria, executed in order |
| `TODO.md` | Working surface — milestones decomposed into single-sitting tasks just-in-time |
| `ARCHITECTURE.md` | Recorded decisions (including UI-motion: Spline 3D vs scroll animation) — never re-litigated mid-loop |
| `API.md` | Endpoint reference, updated whenever an API surface changes |
| `CHANGELOG.md` | Clean, user-facing record of what shipped |
| `EXECUTION_LOG.md` | Process journal — decisions, blockers, gate results per task |
| `KNOWN_ISSUES.md` | Open problems, accepted debt, deliberate deferrals |

## Installation

Copy **both** skill folders into your skills directory (they work as a pair):

```
.claude/skills/                 # per-project  — or —  ~/.claude/skills/  # all projects
├── agentic-dev-loop/
└── init-agentic-loop/
```

## Usage

Just ask for autonomous work — the skill triggers on phrases like:

> *"work through the roadmap"* · *"build this autonomously"* · *"keep going until it's done"* · *"act as the dev team on this"*

On a project without the docs, initialization runs first: repo inspection **before** any questions, one batched interview for only the gaps, then a roadmap held to a PM-grade bar (outcome goals, verifiable exit criteria, dependency-ordered, riskiest unknown first) and adversarially reviewed by an independent subagent before you see it. An existing roadmap is audited the same way — corrections are proposed, never silently applied.

To (re)initialize the docs **without** starting the loop:

```
/init-agentic-loop
```

## Built-in brakes

Autonomy without a circuit breaker just means failures compound silently. The loop stops and reports when:

- the same task fails its quality gates **3×** with genuinely different approaches
- a decision has **no clearly better option** (auth provider, breaking API change, irreversible migration)
- it detects **thrashing** (undo/redo of the same change)
- a **milestone completes** — summary posted; pauses if the next milestone raises a user-intent question
- the requested **scope is done** — it never invents work to keep looping

## Subagent role separation

On nontrivial tasks the reviewer never grades its own work:

```mermaid
flowchart TD
    O["Orchestrator<br/><i>plan · docs · commit · loop control</i>"] --> I["Implementer<br/><i>code + run all gates</i>"]
    O --> T["Test-writer<br/><i>tests from spec, not from code</i>"]
    O --> R["Reviewer<br/><i>diff-only audit: security · perf · architecture · UX</i>"]
    T -. tests must pass .-> I
    R -. flags issues for fix pass .-> I
```

## Repository layout

```
agentic-dev-loop/
├── SKILL.md                    # the loop: selection, cycle, stop conditions, subagents
├── references/                 # loaded on demand
│   ├── init-project.md         # inspection + interview + doc synthesis
│   ├── quality-gates.md        # definition of "done"
│   ├── architecture-checklist.md
│   ├── security-checklist.md
│   ├── performance-checklist.md
│   └── frontend-ux.md
└── assets/doc-templates/       # the seven project docs
init-agentic-loop/
└── SKILL.md                    # /init-agentic-loop slash command
```

## License

[MIT](LICENSE)
