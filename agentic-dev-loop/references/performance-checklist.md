# Performance Checklist

Optimize only after correctness — don't let this checklist tempt premature optimization on code whose behavior isn't verified yet.

- **Unnecessary re-renders** (frontend): check for missing memoization or effect dependencies causing wasted renders on hot paths.
- **N+1 queries**: any loop that hits the database/API once per item should be batched.
- **Missing indexes**: new query patterns on large tables should have a matching index; check the query plan if unsure.
- **Unnecessary allocations**: avoid recreating large objects/arrays on every render or request when they could be memoized or hoisted.
- **Duplicated requests**: check for the same data being fetched multiple times in one flow (e.g., no request de-duplication or caching layer).
- **Oversized bundles**: watch for large dependencies pulled in for a small use case; check if a lighter alternative or a more targeted import exists.

If a performance issue is found but fixing it now would expand the task's scope significantly, log it in KNOWN_ISSUES.md with enough detail to act on later rather than scope-creeping the current task.
