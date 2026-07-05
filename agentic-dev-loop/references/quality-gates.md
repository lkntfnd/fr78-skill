# Quality Gates — Definition of Done

A task is not complete until every applicable box is checked. "Applicable" matters — a copy-change doesn't need a performance review, but don't use that as an excuse to skip checks that do apply.

## Always required
- [ ] Builds successfully (actual build command run, not assumed)
- [ ] Lint passes (actual lint command run)
- [ ] Type checks pass (if the stack has a type checker)
- [ ] Existing tests still pass
- [ ] New tests added for new behavior
- [ ] Documentation updated (CHANGELOG.md minimum; others if they changed)
- [ ] No duplicated logic introduced — checked by actually searching for existing equivalents, not assuming none exist
- [ ] Follows existing project conventions (naming, file layout, error handling style)
- [ ] Edge cases considered and either handled or explicitly noted in KNOWN_ISSUES.md
- [ ] Manually reasoned through the change end-to-end (not just "tests are green")

## If the task touches UI
- [ ] Accessibility: keyboard nav, focus states, alt text, contrast, ARIA where semantic HTML isn't enough
- [ ] Responsive behavior checked at reasonable breakpoints
- [ ] Loading/empty/error states all have real designs, not blank screens

## If the task touches an API surface
- [ ] API.md updated: endpoint, method, request shape, response shape, auth requirement, error responses
- [ ] Backwards compatibility considered — is this a breaking change? If yes, that's a stop-condition decision, not a silent one.

## If the task touches data/schema
- [ ] Migration is reversible or the irreversibility is called out explicitly to the user before running
- [ ] Indexes considered for new query patterns

## What "tests pass" means here
Don't write tests that just assert whatever the implementation currently does — write them against the *intended* behavior from the task description. A test suite that only encodes existing behavior won't catch the bug it's supposed to catch. Cover: the happy path, at least one realistic edge case, and one failure/invalid-input case per new function or endpoint. Not every private helper needs a dedicated test — test at the boundary a caller would actually hit.
