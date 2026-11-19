# SiNiHu milestone

Label: wayfinder:map

## Destination

A build-ready spec for the first SiNiHu milestone: self-hosted, presentations only. Every architecture decision the milestone depends on is locked as an ADR. The map is done when someone can start building without asking anything else:

account → connect an AI provider (BYOK) → pick models → describe a deck → Outline → full deck → edit Cards by hand and with AI → change theme and layout → present in the browser → export PDF and PPTX.

## Notes

- Domain: product and software architecture. Use the vocabulary in `GLOSSARY.md`; record lasting trade-offs in `docs/adr/`.
- Every session consults the **grilling** and **domain-modeling** skills.
- Source docs: `documentation/Project.md`, `documentation/brief.md`.
- Standing decisions from charting (2026-11-19):
  - Self-hosted first. Hosted SaaS comes after this milestone, so the architecture should stay tenant-aware.
  - BYOK covers both raw API keys and SDK or subscription access (Copilot SDK, Claude Agent SDK, Codex SDK). The SDK research decides the approach.
  - Presentations only, but the document model must not rule out other formats later.
  - The stack is a locked preference: Angular, Optimus UI, Tailwind, Angular CDK, Tiptap, NestJS, PostgreSQL, Drizzle, BullMQ. Only a blocking research finding reopens it.
  - The milestone includes in-browser **Present mode**.
  - Accounts are local (email and password, first user is admin).
  - Images: upload or URL, plus AI image generation through the user's provider. Image generation is the last item and is dropped first if anything slips.
  - Record token usage per AI call and show simple totals, with estimated cost where prices are known.
- Research findings live in `research/` next to this map. There is no git remote yet, so there are no research branches.

## Decisions so far

<!-- one line per closed ticket: [title](issues/NN-slug.md): gist -->

- [Gamma as reference behaviour](issues/05-gamma-reference.md): Gamma 5 makes each deck either fluid or fixed 16:9, with grid layouts plus opt-in freeform boxes. Only fixed slides are promised to match in PDF/PPTX export, and Gamma lists its own export gaps. AI edits target the deck, a card or selected elements. The public API exposes text plus an outline, not JSON.
- [The Angular UI stack: Optimus UI and Tiptap](issues/03-angular-ui-stack.md): the stack stands. Optimus UI is `@openng/optimus-ui`, an MIT fork of PrimeNG v21 targeting Angular 22 with a Tailwind plugin. Tiptap's Angular support is the community package `ngx-tiptap` v15 (single maintainer; we build our own toolbars). Pin Angular 22, Optimus 2 and ngx-tiptap 15, and wrap Tiptap behind our own layer.
- [Exporting a JSON document model to PPTX and PDF](issues/04-export-pptx-pdf.md): PptxGenJS is the main Node PPTX writer, and like every PPTX writer it needs absolute boxes. Chromium print-to-PDF works with a flowing layout and gives vector text with embedded fonts. Editable PPTX from a flowing or hybrid Card needs a browser measurement pass (the dom-to-pptx approach).
- [Direct provider APIs and abstraction libraries](issues/02-direct-provider-apis.md): all six providers offer JSON-schema structured output and SSE streaming, with differing schema dialects and partial-JSON streaming. Anthropic has no image generation. Every provider has a key check that generates no tokens, and only OpenRouter and xAI report cost. The Vercel AI SDK (Apache-2.0, v7) is the only library checked that covers all seven targets and has a per-task model registry.
- [Agent SDKs: Copilot, Claude Agent and Codex](issues/01-agent-sdks.md): all three are agent runtimes that start a CLI process per session and keep local disk state. Copilot SDK (MIT) is the only one built for multiple providers: BYOK with no GitHub login, and the only officially allowed sign-in with a user's own subscription (GitHub OAuth). Anthropic and OpenAI restrict subscription login in third-party or hosted apps. Structured output is solid in Claude and Codex but still in preview in Copilot. Image generation through the SDKs is weak or unconfirmed.
- [The Card model: flowing, fixed canvas or hybrid](issues/06-card-model.md): a fixed 16:9 Frame (fluid allowed later); a Layout of flat Regions that the AI fills; reversible Free Elements; overflow warnings, never silent splits; a Chromium render service for PDF and measured PPTX via PptxGenJS (charts as images). See ADR 0001.

## Not yet specified

- **Theme and layout system details**: how themes, typography and color are tokenized, how layout presets bind to Cards, and how AI design variations work. Depends on the document model schema.
- **Editor interactions and AI chat UX**: reordering Elements within and between Regions, pulling out and snapping back Free Elements, resizing, selection, Tiptap's scope inside text Elements, where AI chat lives (deck, Card or selected Elements), and how the overflow warning and its actions appear.
- **Theme constraints for export**: Themes may only use effects and fonts that survive PPTX export (a curated font set), and each Theme sets the minimum text size for auto-shrink.
- **Persistence, autosave and undo**: the save granularity, the undo stack (client vs server), and conflicts between AI edits and manual edits.
- **Repo layout and deployment**: monorepo shape, Docker Compose services (Postgres, Redis for BullMQ, an S3-compatible store such as MinIO), and configuration.
- **Present mode details**: transitions, speaker notes, presenter view.
- **Auth federation**: GitHub, Google and Microsoft. May come forward if the Copilot SDK needs GitHub login.

## Out of scope

- Other formats: documents, carousels, landing pages, infographics. The model must allow them; this map doesn't specify them.
- Importing PDF, PPTX or DOCX.
- Real-time collaboration (Yjs), teams and permissions.
- Hosted multi-tenant SaaS, Stripe, custom domains.
- Brand kits and custom fonts.
- Version history beyond undo/redo and autosave.
- PNG export per Card.
- Public or private share links and published web decks.
