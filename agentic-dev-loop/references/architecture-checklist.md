# Architecture Checklist

## Separation of concerns
Keep these layers distinct — a change in one shouldn't require touching the others unnecessarily:
- UI (presentation only — no business logic in components)
- Business logic (pure where possible, framework-agnostic)
- API layer (routing, request/response shaping, auth checks)
- Data access (queries live here, not scattered in business logic)
- Infrastructure (config, env, deployment)

## Before writing new code
1. Inspect the repo structure — don't assume conventions, read them off existing files.
2. Understand what depends on the code you're about to touch (search for imports/usages).
3. Identify every module the change actually affects — don't scope narrower than the real blast radius.
4. Find the smallest safe implementation that fits existing patterns — prefer extending an existing abstraction over adding a parallel one.

## Anti-patterns to catch (in your own work and in review)
- Giant files (if a file is doing UI + logic + data access, that's a split waiting to happen)
- Giant functions (if you can't name what it does in one short sentence, split it)
- Circular imports
- Hidden side effects (a function named `getX` that also writes to disk or mutates global state)
- Magic values (numbers/strings with meaning that isn't obvious at the call site — name them)
- Duplicated logic — the single most common thing to introduce by accident when moving fast; always search before writing

## Reuse over duplication
Before implementing anything, search for existing:
- Utility functions with similar names or purposes
- Shared components/hooks that do something close to what's needed
- Existing validation, error-handling, or formatting logic

If something close exists but isn't quite right, prefer extending/parameterizing it over forking it, unless the abstraction would become contorted trying to serve both cases — in that case, split cleanly and note the decision in ARCHITECTURE.md.
