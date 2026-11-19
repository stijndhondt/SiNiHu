# The Angular UI stack: Optimus UI and Tiptap

Type: research
Status: resolved
Blocked by:

## Question

What is Optimus UI? Cover who maintains it, what it covers, which Angular version it supports, how mature it is, its license, and how it fits with Tailwind.

What is the current state of Tiptap's Angular integration (official or community, maintenance, gaps)?

Does either finding block the locked stack preference?

## Answer

Full findings: [../research/angular-ui-stack.md](../research/angular-ui-stack.md)

- **Optimus UI** is `@openng/optimus-ui` from the OpenNG org. It is an MIT-licensed community fork of PrimeNG v21, created after PrimeNG 22+ moved to a commercial license. It has more than 80 components and is API-compatible with PrimeNG v21. Other packages named "Optimus" are React, Vue or Flutter, so they are not relevant.
- **Versions and maturity:** v2 targets Angular 22 and v1 targets Angular 21. The first release was in July 2026 and the last npm release was 2.0.2 on 2026-08-28. The repo is still active. It is community-only, with no SLA.
- **Tailwind:** first-class support through `@openng/optimus-ui-tailwindcss`, for Tailwind v3 and v4. It uses CSS layer ordering and token utilities, and also has an unstyled mode with pass-through.
- **Tiptap + Angular:** there is no official package. Tiptap lists `ngx-tiptap` as a community effort. Version 15.0.0 (2026-10-02) supports Angular 22+ and Tiptap ^3.16. It renders node views as Angular components (`AngularNodeViewRenderer`). Its gaps are a single maintainer, slow releases, and Tiptap's UI Components being React-only.
- **Verdict:** nothing blocks the stack. Keep the stack preference. Pin to Angular 22 + Optimus 2 + ngx-tiptap 15, keep Tiptap behind a thin wrapper, and build the editor chrome in Angular.
