# Security Checklist

Run through whatever is applicable to the surface area of the current task — not every task touches auth, but every task touching user input or external data needs the validation/injection items reviewed.

- **Authentication**: is the endpoint/action correctly gated, and is the check on the server side (never trust a client-side check alone)?
- **Authorization**: does the authenticated user actually have permission for *this* resource (not just "is logged in")? Watch for IDOR-style bugs (user A can access user B's record by changing an ID).
- **Input validation**: validate shape, type, and bounds on every external input, server-side, before use.
- **SQL injection**: parameterized queries / ORM only — never string-concatenated queries with user input.
- **XSS**: output encoding on anything rendered from user-controlled data; avoid `dangerouslySetInnerHTML`-style escape hatches unless the content is sanitized.
- **CSRF**: state-changing endpoints need CSRF protection or equivalent (SameSite cookies, tokens) if session-based auth is in use.
- **Rate limiting**: endpoints that are expensive or abusable (auth, search, anything hitting a paid external API) should have limits.
- **Secrets handling**: no secrets in source, logs, or error messages; use environment variables / a secrets manager.
- **Environment variables**: confirm required ones are documented (e.g., in a `.env.example`) without leaking actual values.
- **Secure defaults**: new config should default to the safer option (deny by default, HTTPS, secure cookies) rather than requiring opt-in to safety.

If a task can't reasonably satisfy one of these (e.g., a prototype with no auth system yet), note it explicitly in KNOWN_ISSUES.md rather than silently skipping it.
