# Frontend & UX Standards

Applies whenever a task touches UI.

- Prefer a professional, considered design over a bare-minimum one — but "professional" means restrained and consistent with the project's existing design system, not maximalist. Don't invent a new visual language per feature.
- Motion should serve orientation and feedback (state changes, transitions between views), not decoration for its own sake. A well-timed single transition beats a flashy multi-step animation that adds latency to a common action.
- Scroll-based reveals are appropriate for marketing/landing content; for application UI (dashboards, forms, tools people use repeatedly), prioritize speed and predictability over scroll choreography — the checklist below still applies to core app UI.
- Every interactive element needs visible states: default, hover, focus, active, disabled, loading, error.
- Every async action needs a loading state and an error state designed on purpose, not a blank screen or a raw stack trace.
- Keyboard navigation and focus order should work without a mouse.
- Check contrast ratios on new color combinations, not just "does it look fine on my screen."
- Respect `prefers-reduced-motion` for anything beyond a subtle fade/transition.

This skill aims for a polished, professional feel on genuinely user-facing marketing/narrative surfaces — but polish never overrides the core app UI's need for speed and clarity, and never becomes an excuse to skip the states/accessibility items above in favor of visual polish.

## 3D scenes (Spline) — architecture decision, not a default

Whether a project uses real Spline 3D scenes is decided once, during init (`references/init-project.md`), and recorded in ARCHITECTURE.md. Never assume either way mid-loop:

- **If the user is providing real Spline scenes**: they'll supply the scene export/embed (typically a published Spline URL, or a `.splinecode` file for `@splinetool/react-spline` / the `<spline-viewer>` web component). Only integrate scenes actually provided — never fabricate a Spline URL or fake an embed. Treat each scene as an asset with a real dependency: lazy-load it (it's typically heavy), always ship a static poster-image fallback for slow connections and for `prefers-reduced-motion`, and never let it block first paint of surrounding content.
- **If no Spline scenes are provided**: don't approximate one with a fake iframe or placeholder — build the "modern, professional" feel with the scroll-animation approach below instead. This is the default path unless the user opted into Spline during init.

## Scroll-based motion (default, when no Spline scene is in use)

For marketing/landing pages and other narrative surfaces, professional scroll-driven motion is the default look this skill aims for:

- Use scroll-triggered reveals (fade/slide-in as sections enter viewport) via `IntersectionObserver` or a small motion library already in the stack (Framer Motion, GSAP ScrollTrigger) — don't hand-roll scroll-position math if the project already has one of these.
- Sequence reveals so the page tells a story top-to-bottom (one clear idea per section beat) rather than animating everything at once.
- Real imagery and SVG illustrations belong here, not just abstract shapes — source real photos via an image generation/search tool if one is available in the environment; otherwise use assets already in the repo or note the missing asset in KNOWN_ISSUES.md — never invent an asset URL. Use crisp, purpose-drawn SVGs (icons, illustrative diagrams) rather than generic clipart-style stock graphics.
- Keep it performant: animate `transform`/`opacity` only (not layout-triggering properties), debounce/throttle scroll listeners, and always respect `prefers-reduced-motion` by falling back to a simple fade or no animation at all.
- This still has to satisfy the loading/empty/error-state and keyboard-navigation requirements above — a beautiful scroll experience that traps keyboard focus or has no accessible fallback fails the quality gate, not just the aesthetic bar.
