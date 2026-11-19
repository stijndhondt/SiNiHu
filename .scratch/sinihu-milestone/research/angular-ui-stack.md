# Angular UI stack: Optimus UI and Tiptap

Research for ticket [03-angular-ui-stack](../issues/03-angular-ui-stack.md). Checked 2026-11-19 against npm, GitHub and official docs. Current Angular is 22.3.0 (`@angular/core` on npm).

## 1. What "Optimus UI" is

### Candidates found under that name

| Package | Framework | Status | Relevant? |
|---|---|---|---|
| `@openng/optimus-ui` (OpenNG) | Angular | Active, v2.0.2 | **Yes, this is the one the stack means** |
| `optimus-ui` (Tripplo Software) | React (peer `react >=16`, uses antd + styled-components) | Last published 0.0.6, 2021; metadata modified 2022 | No |
| `@optimuscms/ui` | Vue + Bulma | 1.1.10, last modified 2022 | No |
| `optimus` on pub.dev | Dart/Flutter | n/a | No |

Sources: npm search https://registry.npmjs.org/-/v1/search?text=optimus%20ui ; https://www.npmjs.com/package/optimus-ui ; https://www.npmjs.com/package/@optimuscms/ui ; https://pub.dev/documentation/optimus/latest/optimus/

### `@openng/optimus-ui`: the Angular library

- **What it is:** A community fork of **PrimeNG v21**, the last MIT-licensed PrimeNG. The README says: "In June 2026 PrimeTek ... moved future major versions, starting with v22, to a commercial license. ... Optimus UI is a community fork of PrimeNG v21 ... It stays MIT. There is no paid tier and none is planned." It is API-compatible with PrimeNG v21. It keeps the `p-*` selectors (`<p-button>`) and ships a `migrate-from-primeng` schematic. https://github.com/openng-org/optimus-ui ; https://www.npmjs.com/package/@openng/optimus-ui
  - PrimeTek's own page confirms the reason: PrimeNG 22+ moves to the "PrimeUI license" and "will no longer be released as open source". It adds that "all existing MIT-licensed versions remain MIT forever". The free Community tier is for organizations under $1M revenue and fewer than 5 developers. Commercial use costs $599 per developer. https://primeui.dev/nextchapter
  - Discrepancy: the Optimus README says PrimeTek "archived the PrimeNG repository". The GitHub API reports `primefaces/primeng` as `archived: false`, last pushed 2026-09-24. `primeng@22.1.5` (2026-11-11) is on npm with `SEE LICENSE IN LICENSE.md`. The license change is real, but the "archived" wording is not accurate.
- **Maintainer:** The OpenNG org (https://www.openng.org/). The npm maintainers are `ggrignon` and `dominicbachmann`. The GitHub contributor list shows PrimeTek's historical committers because the full history was carried over. The README says: "Not affiliated with, endorsed by, or sponsored by PrimeTek." Support is "Community only — there is no SLA and no commercial support."
- **Components:** "80+ accessible components", which is the PrimeNG v21 set (data table, tree, forms, overlays, menus, paginator, and so on). Themes ship as Aura, Material, Lara and Nora presets in `@openng/optimus-ui-themes`. There is a styled mode (CSS variables) and an unstyled mode, and icons come from `@openng/icons`. Related packages are `-styled`, `-styles`, `-motion`, `-utils`, `-locale` and `-tailwindcss`.
- **Angular versions:** The compatibility table at https://optimus.openng.org/installation lists **Optimus v2 → Angular 22** and **v1 → Angular 21**. One Optimus major targets one Angular major. The npm `latest` (2.0.2) has peer `@angular/core ^22.1.4` and `@angular/cdk ^22.1.4`. The `v1-lts` tag is 1.0.2.
- **Release cadence and last release:** The first npm publish was 2026-07-21. Twelve versions came out between July and August: 1.0.0 on 2026-08-03, 2.0.0 on 2026-08-18, and **2.0.2 on 2026-08-28, which is the last release**. There has been no npm release for about 12 weeks. The repo is still active: last push 2026-11-19, 21 PRs merged since 2026-10-01, 815 stars, and 978 open issues (most likely carried over from PrimeNG; UNVERIFIED). Downloads were about 245k in the last month (https://api.npmjs.org/downloads/point/last-month/@openng/optimus-ui). It is already used downstream, for example `@garudalinux/core` ("based on Optimus UI").
- **License:** MIT. LICENSE.md keeps both the OpenNG 2026 and PrimeTek 2016–2026 copyrights. Note that the GitHub API reports "Other/NOASSERTION" only because of the dual-copyright file. https://github.com/openng-org/optimus-ui/blob/main/LICENSE.md
- **Tailwind fit:** First-class, through `@openng/optimus-ui-tailwindcss` (peer `tailwindcss >=3.1.0`). One package covers Tailwind v4 (`@import "@openng/optimus-ui-tailwindcss";`) and v3 (`plugins: [OptimusUiTailwind]`). In styled mode it exposes theme tokens as utilities (`bg-primary`, `text-surface-500`). It recommends CSS layer ordering (v4: `order: 'theme, base, optimus'`) over `!important`, and you need to align `darkModeSelector` with Tailwind's dark variant. Unstyled mode plus pass-through lets you style components entirely with Tailwind. https://optimus.openng.org/tailwind
- **Unverified:** How fast it will support Angular 23 (expected around mid-2027), and the long-term health of a fork that is 5 months old. Its roadmap at https://optimus.openng.org/roadmap was not reviewed in detail.

### Alternatives (for comparison if Optimus is rejected)

| Library | Latest | Angular peer | License | Tailwind fit |
|---|---|---|---|---|
| PrimeNG (PrimeTek) | 22.1.5, 2026-11-11 | ^22.1.0 | Commercial "PrimeUI license" from v22 (free Community tier with limits) | Same tailwind plugin model as Optimus (Optimus inherited it) |
| Spartan/ui (`@spartan-ng/brain`) | 1.9.0, 2026-10-29 | >=21 <23 | MIT | Built for Tailwind (shadcn-style, copy-in helm components) |
| Angular Material (Google) | 22.3.0, 2026-11-18 | ^22 \|\| ^23 | MIT | Works alongside Tailwind, but its theming is Sass/CSS-token based, so it is a looser fit |

Sources: https://www.npmjs.com/package/primeng ; https://www.npmjs.com/package/@spartan-ng/brain ; https://www.npmjs.com/package/@angular/material

## 2. Tiptap's Angular integration

- **There is no official Angular package.** `@tiptap/angular` does not exist on npm (404). Tiptap's install page lists JS, React, Next, Vue 2/3, Nuxt, Svelte, Alpine and PHP as official. **Angular appears only under "Community efforts"** and links to `ngx-tiptap`. https://tiptap.dev/docs/editor/getting-started/install
- **Community package `ngx-tiptap`** (https://github.com/sibiraj-s/ngx-tiptap, https://www.npmjs.com/package/ngx-tiptap):
  - MIT license, about 344k downloads in the last month, 203 stars, 0 open issues, not archived.
  - **Effectively a one-person project.** `sibiraj-s` has 325 commits; the next human contributor has 2.
  - Release history: 13.0.0 (2025-07, Tiptap 2), 14.0.0/14.0.1 (2025-08, Tiptap 3), then nothing until **15.0.0 on 2026-10-02**. That is roughly a 14-month gap, which suggests releases follow Angular majors rather than a regular cadence.
  - Compatibility: 15.0.0 has peers `@angular/* >=22.0.0`, `@tiptap/core`/`@tiptap/pm` `^3.16.0`, `@tiptap/extension-bubble-menu`/`floating-menu ^3.16.0` and `@floating-ui/dom ^1`. 14.x covers Angular 20–21 with Tiptap 3. Tiptap itself is at **3.34.0** (2026-11-13) and releases every week. https://www.npmjs.com/package/@tiptap/core
  - Features in the README: a standalone `<tiptap-editor>` with `ngModel`, reactive forms **and Signal Forms**; `<tiptap-bubble-menu>` and `<tiptap-floating-menu>`; **`AngularNodeViewRenderer`**, which renders Angular components as node views (using `Injector`, `NodeViewContent`, and the `tiptapDraggable` drag handle directive).
- **Known gaps:**
  - Node views as Angular components **are** supported, but only through the community `AngularNodeViewRenderer`. Tiptap does not maintain it or test against it. Parity with `ReactNodeViewRenderer` features (such as mark views and per-update props) is UNVERIFIED.
  - **Tiptap UI Components and templates (Simple Editor and others) are React-only.** The docs describe them as "React components" built on hooks such as `useTiptapEditor`. https://tiptap.dev/docs/ui-components/getting-started/overview . An Angular app must build its own toolbar, menus and slash commands.
  - Bus factor of one, plus a slow release cadence. If Angular moves faster than ngx-tiptap, the fallback is to use `@tiptap/core` directly. It is framework-agnostic (`new Editor({ element })`), and a thin in-house wrapper is small.
  - Whether Tiptap's paid or cloud extensions (Collaboration, AI, Comments) work under Angular has not been tested. They are core extensions, so they should, but this is UNVERIFIED.

## 3. Does either finding block the stack?

No.
- Optimus UI is a real, MIT-licensed Angular library with Tailwind support and current Angular 22 support. The risk is its age: a fork that is 5 months old, maintained by the community, with no release since August. Because it is PrimeNG-v21-compatible, there are exit routes: back to commercial PrimeNG, or to another library.
- Tiptap works with Angular today through ngx-tiptap (Angular 22, Tiptap 3), including Angular component node views. The gaps (React-only UI kit, single maintainer) cost extra work but do not stop the project.

## Implications for SiNiHu

- **Keep the stack preference. There is no need to reopen it.** Pin to **Angular 22 + `@openng/optimus-ui@2` + `ngx-tiptap@15` + `@tiptap/*@^3.16`**. All three agree on Angular 22.
- **Upgrades will be gated:** each Optimus major tracks exactly one Angular major, and ngx-tiptap moves on its own schedule. Expect Angular upgrades to wait on both libraries. Record this in an ADR.
- **Build our own editor chrome.** The slide toolbar, bubble and slash menus, and block handles have to be written in Angular, because Tiptap's UI Components are React-only. Use Optimus overlays and menus for these.
- **Keep Tiptap behind a seam.** Put a thin internal editor module around `@tiptap/core` so we can replace ngx-tiptap with an in-house wrapper if it stalls. Put custom slide blocks into node views through `AngularNodeViewRenderer`.
- **Tailwind integration:** use `@openng/optimus-ui-tailwindcss` with Tailwind v4, set the CSS layer order `theme, base, optimus`, and use one dark-mode selector shared by Optimus and Tailwind. Consider unstyled mode with pass-through if slide themes must be fully Tailwind-driven.
- **Do not use commercial PrimeNG 22+ by accident.** Imports must come from `@openng/optimus-ui/*`, not `primeng/*`. A lint rule would enforce this.
- **Watch item:** check Optimus release activity before the Angular 23 upgrade (expected around mid-2027). If it has stalled, Spartan/ui (MIT, Tailwind-native) is the closest fallback.
