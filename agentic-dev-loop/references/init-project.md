# Initializing a New Project

Run this before the first cycle of the main loop — and again on demand (via `/init-agentic-loop`) to re-initialize or refresh the docs later. Goal: end up with accurate ROADMAP.md, TODO.md, ARCHITECTURE.md, API.md, CHANGELOG.md, EXECUTION_LOG.md, KNOWN_ISSUES.md that reflect the *actual* repo and the user's *actual* intent — not templates filled with guesses.

Init resolves the real unknowns (scope, architecture) up front, then flows directly into the first task of the loop's first step (when invoked from the main loop — the /init-agentic-loop command stops at the summary instead; see Step 4). It only pauses on something genuinely still unresolved after Steps 1-2 — see Step 4.

## Step 1 — Inspect before asking anything

Don't ask the user things you can determine yourself. Check, in order:

1. **Is this an empty/new repo or an existing codebase?**
   - Determine (using whatever shell/tools the environment provides): is this a git repo at all — if not, stop and ask; the main loop requires version control, so ask the user whether to `git init`. Then classify by the **working tree, not commit count**: empty/new means no meaningful project files exist (nothing beyond e.g. `.git`, a README, a license). A repo with real source files but zero commits is an *existing codebase with uncommitted work* — inspect it as existing code per item 2, and offer the user an initial commit of the existing code before the loop starts, so there's a rewind point that predates any changes the loop makes. List top-level contents, check for a README.
2. **If existing code:**
   - Language(s), framework(s), package manager — read `package.json`/`pyproject.toml`/`go.mod`/`Cargo.toml`/etc., don't guess.
   - Existing folder structure and apparent architectural pattern (MVC? layered? monorepo?).
   - Existing test setup — what framework, current coverage if measurable, do tests currently pass?
   - Existing lint/format/typecheck config — what commands actually run (`package.json` scripts, `Makefile`, CI config).
   - CI/CD setup if any (`.github/workflows`, etc.) — what gates already exist there.
   - Any existing docs (README, existing ROADMAP/ARCHITECTURE/etc. even if informal) — read them, don't overwrite silently.
   - Skim git log for recent activity to understand what's actively being worked on vs. stable/legacy.
   - Grep for `TODO`/`FIXME`/`XXX` comments in code — these are candidate KNOWN_ISSUES.md entries and sometimes TODO.md task entries.
   - Any obvious existing API surface (routes/controllers) — candidate API.md content.
3. **If empty/new repo:** there's nothing to inspect except the user's intent — skip to Step 2 with more weight on the interview.

Summarize what you found in a few lines before asking anything, so the user can correct misreadings rather than you asking questions you could've answered yourself.

## Step 2 — Ask only what you couldn't determine

Batch this into one message, not a drawn-out interrogation. Skip any question the repo/README/user's original request already answered. Typical gaps:

- **Goal & users**: what is this project for, who uses it? (Often answerable from README — confirm rather than ask from scratch if so.)
- **Scope for v1 / current milestone**: what's must-have vs. nice-to-have vs. explicitly out of scope right now?
- **Constraints**: any fixed tech choices, deployment target, compliance/regulatory requirements, timeline pressure that should affect prioritization?
- **Definition of done**: what does "this milestone is complete" actually mean to the user — a specific demo, a deployed environment, passing a specific test suite?
- **Architecture & stack** — for a new project, or when inspection surfaced a genuinely open fork: does the user have a preferred framework, database, service shape, or deployment target? If they have no preference, propose one in Step 3's ARCHITECTURE.md as a decision flagged for their confirmation. Either way, what's decided gets recorded there so it isn't re-litigated mid-loop.
- **Branch strategy** — where should the loop's per-task commits land: directly on the current branch, or on a dedicated working branch (e.g. `agent/<milestone>`) merged back at each milestone check-in? Recommend the dedicated branch whenever the repo has a remote or CI/CD watching the main branch — rejecting a whole run stays one command instead of unwinding dozens of commits; direct commits are fine for a solo/local project. Record the choice in ARCHITECTURE.md. Either way the loop never pushes — pushing stays a user action.
- **Continuation policy** — when the roadmap completes, should the loop stop and propose the next phase (default), or scope the next phase itself and keep going (**continuous mode** — see "Extension mode" below and SKILL.md §9)? Recommend continuous mode only for projects the user frames as open-ended. Record the choice in ARCHITECTURE.md.
- **Anything already flagged as broken or risky** that isn't obvious from the code (e.g., "the payments module is fragile, be careful there").

Don't ask about things covered by SKILL.md's fixed defaults (priority order, quality gates, mindset) unless the user's context suggests a real exception (e.g., a frontend-only project makes the DB-first priority moot — note that rather than asking).

## Step 3 — Synthesize into the docs

Copy `assets/doc-templates/*` into the project root (for any doc that already exists with real content, don't overwrite it — keep its content and fold in the template's missing structural elements, e.g. per-milestone `Status:` lines, so the loop can operate on it).

- **ROADMAP.md**: 2-5 milestones max for a first pass, derived from the stated goal and scope — not an exhaustive feature list. Order matters: the loop works through these in sequence, so put them in the order they should actually happen (respecting the Database → Backend → Auth → APIs → Frontend → Polish → Optimization default from SKILL.md unless there's a reason to override it). Hold it to a real project manager's bar — the loop will execute this document milestone-by-milestone, so its quality caps the quality of everything downstream:
  - Each milestone's **Goal** is an outcome ("what's true after this that isn't true now"), not a list of activities.
  - Each milestone has verifiable exit criteria the loop can actually check — tied to the user's stated definition of done, not vague ("auth works" → "a user can sign up, log in, and reach a protected page; test suite covers all three").
  - Sequencing respects real dependencies, and the riskiest assumption or hardest unknown is de-risked in an early milestone, not parked at the end.
  - Nothing the user named as must-have is missing; nothing they scoped out has crept in.
  - **Review pass before presenting**: run one adversarial review of the draft roadmap against the four points above — as a dispatched reviewer subagent when available (give it only the inspection summary, the user's interview answers, and the draft — not your drafting reasoning), otherwise as an explicit separate self-review pass. Fold accepted critiques in before showing the user.
  - **If a ROADMAP.md with real content already exists**: review it against this same bar and against what inspection actually found. Propose corrections to the user (stale milestones, already-shipped work still listed as open, missing dependency ordering) rather than silently rewriting their roadmap — or silently executing a roadmap you can see is wrong.
- **TODO.md**: create one placeholder section per ROADMAP.md milestone, in the same order, all `STATUS: NOT STARTED` — then immediately decompose only the *first* milestone into real, concrete, single-sitting tasks and flip it to `STATUS: IN PROGRESS` (in both TODO.md and ROADMAP.md, so the two docs never disagree about the active step). Do not decompose later milestones yet; that happens just-in-time when the loop reaches them. Also fold in any setup gaps found in Step 1 (no tests configured, no lint config, no CI) as early tasks under the first step if they block real work.
- **ARCHITECTURE.md**: for existing code, document what's actually there (real folder structure, real patterns observed) — don't prescribe an idealized architecture that doesn't match reality. For a new project, propose one and flag it as a decision the user should confirm, not silently assume. Record the key decisions from Step 2 here as architecture decisions, not just conversation history — so they don't get silently re-litigated mid-loop.
- **API.md**: document any existing endpoints found in Step 1. Empty/skeleton for a new project.
- **CHANGELOG.md**: seed with one entry noting the init date and what was found/decided.
- **EXECUTION_LOG.md**: seed with one entry documenting the init pass itself — what was inspected, what was asked, what was decided — so the process record starts from init, not from the first task.
- **KNOWN_ISSUES.md**: seed with anything found in Step 1 (TODOs, failing tests, missing CI, fragile areas the user flagged).

Once the docs are synthesized, commit them as a dedicated init commit (e.g., "docs: initialize project docs (agentic-dev-loop init)") before any code work begins — on an empty repo this is the repo's first commit, and either way it gives the loop a clean rewind point that predates all task commits. Stage only the docs init created or edited — never sweep the user's uncommitted files into this commit; pre-existing uncommitted code is handled per Step 1 (a separate, user-approved initial commit).

## Step 4 — Confirm only what's genuinely a major, unresolved decision

Don't pause by default. Any real architecture fork was already resolved in Step 2's interview — if it's settled, proceed straight into the first task of the loop's first step rather than stopping to ask again.

Pause and wait for the user only when something major is still genuinely unresolved after inspection + interview — most commonly: an architecture choice for a brand-new repo that the interview didn't pin down, a scope ambiguity where two fundamentally different implementations are equally valid, or proposed corrections to an existing ROADMAP.md (Step 3) that the user hasn't accepted or declined yet — never decompose the first milestone or start executing against a roadmap whose corrections are pending. In those cases, present the options and wait.

Otherwise: show a brief summary (milestones, first step's task breakdown, key decisions recorded) as you go, and begin executing — don't make the user say "go" for a decision that's already been made.

**How this ends depends on how it was invoked:**

- **Via the `/init-agentic-loop` slash command**: stop after presenting the summary (milestones, first step's task breakdown, key decisions recorded). Do NOT begin task execution unless the user says to.
- **From the main loop (agentic-dev-loop SKILL.md §0)**: proceed straight into the first task as described above.

## Extension mode — planning the next phase

Run this instead of the full init when the project's docs already exist and every ROADMAP.md step is DONE (or the user asks to plan the next milestones). No full interview — the finished project itself is now the interview:

1. **Review like a PM with full context**: the recorded goal, users, and decisions in ARCHITECTURE.md; everything in KNOWN_ISSUES.md (open problems, accepted debt, deferred items — deferrals often name their own "what would trigger revisiting"); the CHANGELOG.md delta since the last phase; and the actual repo state (gaps between what the docs promise and what exists).
2. **Derive 2-5 next milestones** from that evidence: unresolved issues worth promoting to milestones, deferred scope whose trigger has arrived, and the natural next increment toward the recorded product goal. Every milestone must trace to a recorded source — never invent scope orthogonal to the project's stated intent. A genuine product-direction fork (new user segment, monetization, a pivot) is a stop-and-ask even in continuous mode.
3. **Hold them to the same bar as the first roadmap** — Step 3's quality criteria and the same adversarial review pass before finalizing.
4. **Append, don't rewrite**: add the new milestones to ROADMAP.md as `NOT STARTED` below the DONE ones, add matching placeholder sections to TODO.md, and log the extension pass in EXECUTION_LOG.md (what was reviewed, what was derived, from which sources).
5. Then follow the continuation policy recorded in ARCHITECTURE.md: **continuous** → decompose the first new milestone and keep the loop going; **stop-and-propose** (or invoked via `/init-agentic-loop`) → present the proposed phase and wait.
