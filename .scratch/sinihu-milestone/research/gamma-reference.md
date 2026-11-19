# Gamma as reference behaviour

Research for ticket [05-gamma-reference](../issues/05-gamma-reference.md). Sources fetched 2026-11-19. Gamma is reference behaviour only; SiNiHu builds its own architecture and look.

**Important context:** Gamma now runs two editors. **Gamma 5** (current) rebuilt the layout model. **Classic** gammas, made before Gamma 5, "open unchanged in the classic editor" ([grids & freeform](https://help.gamma.app/en/articles/17572896-how-do-grids-and-freeform-editing-work-in-gamma)). Gamma 5 help pages mostly say "slide" where Classic pages say "card". The API still counts "cards".

Sources are Gamma's own help center (help.gamma.app) and developer docs (developers.gamma.app). I did not find a separate `gamma.app/docs` site. Anything marked **UNVERIFIED** is not stated in a primary source.

## 1. Card height: fluid vs fixed

- **Classic:** "By default, cards are fluid, meaning they automatically resize based on the content inside them." You set a minimum aspect ratio in ⋯ → Page Setup. Choosing a fixed ratio (e.g. 16:9) shows dotted guidelines ([cards article, search snippet](https://help.gamma.app/en/articles/11016396-what-are-cards-in-gamma-and-how-to-do-they-work); [page setup](https://help.gamma.app/en/articles/11029115-how-do-i-control-slide-sizes-and-adjust-page-setups)). The cards article has since been rewritten for Gamma 5. The Classic wording above comes from a search-engine snippet of that page, so it is close to primary.
- **Gamma 5:** fluid vs fixed is a **per-gamma** setting. Fixed slides "keep their 16:9 size and scale the content down to fit, down to a minimum readable size". Fluid slides keep content at its size and grow taller, like a document page ([grids & freeform](https://help.gamma.app/en/articles/17572896-how-do-grids-and-freeform-editing-work-in-gamma)). Presentations use a "fixed 16:9 canvas" ([cards article](https://help.gamma.app/en/articles/11016396-what-are-cards-in-gamma-and-how-to-do-they-work)).
- Page Setup → Slide Size offers fixed-height options. Gamma recommends fixed height "to maintain consistency" and for PowerPoint and social exports ([page setup](https://help.gamma.app/en/articles/11029115-how-do-i-control-slide-sizes-and-adjust-page-setups)).
- The **API** (Classic) `cardOptions.dimensions` values per format ([Generate API parameters](https://developers.gamma.app/guides/generate-api-parameters-explained)):
  - presentation: `fluid` (default), `16x9`, `4x3`
  - document: `fluid` (default), `pageless`, `letter`, `a4`
  - social: `1x1`, `4x5` (default), `9x16`
  - The Gamma 5 API has no such field. The size comes from the chosen `templateId` ([migration guide](https://developers.gamma.app/get-started/migrate-from-classic-to-gamma-5)).
- The gap between cards is fixed in both editors. Classic can remove it with Full-bleed. Gamma 5 flexible-height webpage sections have no gap ([page setup](https://help.gamma.app/en/articles/11029115-how-do-i-control-slide-sizes-and-adjust-page-setups)).

## 2. Layouts and positioning

- **Classic:**
  - Content flows top to bottom inside a card.
  - Structure comes from **columns**, **Smart Layouts** (preset templates such as timeline, gallery and columns, inserted from the Insert menu) and **card layouts** ([grids & freeform](https://help.gamma.app/en/articles/17572896-how-do-grids-and-freeform-editing-work-in-gamma); [transform layouts](https://help.gamma.app/en/articles/11029130-what-s-the-fastest-way-to-transform-content-and-layouts)).
  - **Accent image** layouts per card: No image, Top, Left, Right, Background. Background images take an overlay of Frosted, Faded or Clear.
  - Other per-card settings: **Full bleed**, **Content alignment** (top, center, bottom) and **Content width** (medium, large) ([style slides](https://help.gamma.app/en/articles/11969695-how-do-i-style-slides-and-adjust-layout-settings-in-my-gamma)).
  - "Try Layouts" (sparkle icon) swaps among AI-suggested layouts ([transform layouts](https://help.gamma.app/en/articles/11029130-what-s-the-fastest-way-to-transform-content-and-layouts)).
  - **Free positioning in Classic: UNVERIFIED.** I found no Classic article that describes it.
- **Gamma 5:**
  - "Grids are the primary way content is laid out on a slide." Grids replace columns, smart layouts and card layouts with proportional rows and columns plus presets. Grid content reflows in order as you edit.
  - **Freeform** boxes sit at exact canvas positions outside the grid and can be dragged, resized and rotated. Shapes are always freeform. A freeform block cannot go back to the grid, and freeform content does not reflow.
  - On phones, grid sections stack into one column.
  - Sources: [grids & freeform](https://help.gamma.app/en/articles/17572896-how-do-grids-and-freeform-editing-work-in-gamma).
  - Accent image layouts remain: left, right, top or none. Spacing is set by Density (Compact, Comfortable, Spacious) or Custom Spacing ([style slides](https://help.gamma.app/en/articles/11969695-how-do-i-style-slides-and-adjust-layout-settings-in-my-gamma); [page setup](https://help.gamma.app/en/articles/11029115-how-do-i-control-slide-sizes-and-adjust-page-setups)).
  - "Try new layout" is gone. You ask the Agent instead ([transform layouts](https://help.gamma.app/en/articles/11029130-what-s-the-fastest-way-to-transform-content-and-layouts)).
  - In the API, accent images "are placed automatically and can't be controlled". Placement "works best with fixed-dimension formats like 16x9" ([Generate API parameters](https://developers.gamma.app/guides/generate-api-parameters-explained)).

## 3. Overflow

- **Fluid:** there is no overflow because the card grows ([grids & freeform](https://help.gamma.app/en/articles/17572896-how-do-grids-and-freeform-editing-work-in-gamma)).
- **Fixed (Gamma 5):**
  1. Content first scales down to a "minimum readable size".
  2. Past that, content that overflows the 16:9 frame causes a **grey stripe or border**.
  3. Gamma's suggested fixes are manual: duplicate the slide and split the content, or switch the gamma to fluid height ([grids & freeform](https://help.gamma.app/en/articles/17572896-how-do-grids-and-freeform-editing-work-in-gamma)).
  4. No automatic split into new slides is documented. Automatic splitting is **UNVERIFIED / not found**.
- **Classic fixed:** dotted guidelines show the frame ([page setup](https://help.gamma.app/en/articles/11029115-how-do-i-control-slide-sizes-and-adjust-page-setups)). Whether overflow there is clipped, scaled or scrolled is **UNVERIFIED**.

## 4. Present mode and tall cards

- "Use Up/Down arrow keys to scroll through longer slides." Left and Right move between slides. Enter opens toggles and nested cards. Trackpad scrolling previews the next or previous slide. Present runs Full screen or In tab, with a presenter view for notes ([present](https://help.gamma.app/en/articles/8032935-what-s-the-best-way-to-present-my-gamma)). So tall fluid cards **scroll** within the card in Present mode. Whether they are also scaled is not stated.
- Gamma 5 slides "fill the frame edge to edge in Present mode and exports". Classic shows blurred backdrops unless Show Slide Backdrops is on ([page setup](https://help.gamma.app/en/articles/11029115-how-do-i-control-slide-sizes-and-adjust-page-setups)).
- Exports reflect "Present Mode, not Edit Mode" ([export mismatch](https://help.gamma.app/en/articles/15939201-why-doesn-t-my-exported-pdf-or-powerpoint-match-what-i-see-in-gamma)).

## 5. Export to PDF and PPTX

- **Formats:** PDF, PNG, PPTX and Google Slides (web only). There is no .docx. You can export all cards or only the current card ([export](https://help.gamma.app/en/articles/8022861-what-s-the-easiest-way-to-export-my-gamma)). PNG via the API is a zip with **one image per card** ([Generate API parameters](https://developers.gamma.app/guides/generate-api-parameters-explained)).
- **Fixed slides** export "at the same 16:9 frame shown in the editor, so PowerPoint and PDF exports match the screen" ([grids & freeform](https://help.gamma.app/en/articles/17572896-how-do-grids-and-freeform-editing-work-in-gamma)).
- **Zoom to fit** (Share → Export) scales content to fill the page. It "only applies to fixed-format exports" and does not affect PDF or PNG exports with fluid slides ([export mismatch](https://help.gamma.app/en/articles/15939201-why-doesn-t-my-exported-pdf-or-powerpoint-match-what-i-see-in-gamma)).
- **Page splitting of tall fluid cards in PDF and PPTX: UNVERIFIED.** No primary source says whether a fluid card becomes one tall page, splits across pages, or is scaled. The ebook guide only warns that a long table of contents "may span more than one card" and that you should check the PDF ([ebook](https://help.gamma.app/en/articles/16767225-how-do-i-create-an-ebook-in-gamma)).
- **PPTX editability:**
  - Tables export as **editable PowerPoint tables** by default, with an Editable tables toggle. Rounded corners become sharp.
  - Speaker notes are included.
  - The slide master is **blank**.
  - AI images export as PNG.
  - Accent image cutouts are supported in PDF and PPTX.
  - Sources: [export](https://help.gamma.app/en/articles/8022861-what-s-the-easiest-way-to-export-my-gamma).
  - Whether general text exports as editable PowerPoint text boxes is **not stated in the help center: UNVERIFIED**. Gamma's own marketing guides (secondary, promotional) describe the PPTX as editable but say it often needs cleanup after export ([guide](https://gamma.app/explore/content/guides/what-ai-creates-templates-i-can-use-in-powerpoint)).
- **Known fidelity limits Gamma states:**
  - Gradient headings may render as a single colour. Frosted overlays fall back to simpler styles ([export](https://help.gamma.app/en/articles/8022861-what-s-the-easiest-way-to-export-my-gamma)).
  - Fonts:
    - Theme fonts are embedded, with a cap of about 64 MB.
    - Weights outside the theme fall back to the nearest weight.
    - Italic is a slanted regular font, not a true italic.
    - Google Slides ignores embedded fonts.
    - Source: [export](https://help.gamma.app/en/articles/8022861-what-s-the-easiest-way-to-export-my-gamma).
  - Solid page backgrounds from the theme or Page Setup may be missing from PDF, PPTX and Slides exports. The workaround is to set a background colour on each card ([export mismatch](https://help.gamma.app/en/articles/15939201-why-doesn-t-my-exported-pdf-or-powerpoint-match-what-i-see-in-gamma)).
  - Charts and diagrams can come out small or shifted ([export mismatch](https://help.gamma.app/en/articles/15939201-why-doesn-t-my-exported-pdf-or-powerpoint-match-what-i-see-in-gamma)).
  - Google Sheets charts become static images ([export](https://help.gamma.app/en/articles/8022861-what-s-the-easiest-way-to-export-my-gamma)).
  - Heavy images can fail or render incorrectly ([export mismatch](https://help.gamma.app/en/articles/15939201-why-doesn-t-my-exported-pdf-or-powerpoint-match-what-i-see-in-gamma)).
  - Gammas of about 150 or more slides can fail. Mobile blocks PPTX export above 150 slides ([export](https://help.gamma.app/en/articles/8022861-what-s-the-easiest-way-to-export-my-gamma); [export mismatch](https://help.gamma.app/en/articles/15939201-why-doesn-t-my-exported-pdf-or-powerpoint-match-what-i-see-in-gamma)).
  - Classic "Show Slide Backdrops" can cause white borders in PPTX ([page setup](https://help.gamma.app/en/articles/11029115-how-do-i-control-slide-sizes-and-adjust-page-setups)).
  - PDFs are static ([export](https://help.gamma.app/en/articles/8022861-what-s-the-easiest-way-to-export-my-gamma)).
  - Export failure reasons in the API/MCP are `render_timeout`, `deck_too_large`, `no_content` and `export_failed` ([MCP tools](https://developers.gamma.app/mcp/mcp-tools-reference)).
- **Secondary (Gamma community forum, not docs):** users report that content in fixed-size cards shrinks on 16:9 PPTX exports, which leaves dense slides cramped ([forum](https://community.gamma.app/x/api/msg_PUmiHX4ybFXZ/api-enhancement-requests-editable-presentations-an)).

## 6. AI editing scopes

- **Three scopes** in the editor ([edit with AI](https://help.gamma.app/en/articles/8033284-how-do-i-edit-my-content-with-ai)):
  - **Whole gamma:** a chat request such as a tone change. Classic uses the Agent button at the top right.
  - **Single slide or card:** "Edit with Agent" on the slide's left edge.
  - **Selected elements:** the Agent toggle enters AI editing mode. Clicked elements appear in the chat as the target.
- **Review:** an Original/Modified toggle shows before and after, even across slides. Changes from one request are accepted **all together or not at all**, with restore available ([edit with AI](https://help.gamma.app/en/articles/8033284-how-do-i-edit-my-content-with-ai)).
- **Restyle:** "ask for a new look" restyles the whole deck and keeps the content ([style slides](https://help.gamma.app/en/articles/11969695-how-do-i-style-slides-and-adjust-layout-settings-in-my-gamma)).
- **Agent can overwrite:** the Agent can sometimes overwrite an existing slide when asked for a new diagram ([cards article](https://help.gamma.app/en/articles/11016396-what-are-cards-in-gamma-and-how-to-do-they-work)).
- **API and MCP edits:**
  - `POST /v1.0/agent/gammas/{gammaId}/edits` takes a natural-language instruction. Only one edit runs at a time ([API options](https://developers.gamma.app/get-started/understanding-the-api-options)). The help center says it targets one page and cannot add pages ([API help](https://help.gamma.app/en/articles/11962420-does-gamma-have-an-api)).
  - The MCP `edit_gamma` tool takes optional `cardIds` (1–100) to narrow scope. It can add, remove, reorder and rewrite cards, and it returns a `summary` ([MCP tools](https://developers.gamma.app/mcp/mcp-tools-reference)).
  - The two sources disagree on whether an edit can add pages: the help center says no, the MCP reference says yes when asked. **Unresolved.**

## 7. Public API and document representation

- **Public REST API:** "Generate API v1.0", generally available since 5 Nov 2025 on all plans. It authenticates with an `X-API-KEY` header or OAuth 2.0. An MCP server runs at `https://mcp.gamma.app/mcp` ([API help](https://help.gamma.app/en/articles/11962420-does-gamma-have-an-api)).
- **Endpoints:**
  - Generate: Classic `POST /v1.0/generations` and `/from-template`; Gamma 5 `POST /v1.0/agent/generations`.
  - Edit.
  - Export: `POST /v1.0/gammas/{id}/export` for pdf, pptx or png.
  - List and search.
  - Metadata get, comments, analytics, duplicate, archive and delete.
  - Source: [API options](https://developers.gamma.app/get-started/understanding-the-api-options).
- **Document representation:** **no structured card or element schema is exposed.**
  - REST `GET /gammas/{id}` returns metadata only. A generation returns `gammaUrl`, `exportUrl` and a summary ([API options](https://developers.gamma.app/get-started/understanding-the-api-options); [migration guide](https://developers.gamma.app/get-started/migrate-from-classic-to-gamma-5)).
  - MCP `read_gamma` returns **formatted text** ("not raw HTML or JSON"): a `Title:` line, then `=== Page N: <title> ===` per page.
  - MCP `get_gamma_outline` returns pages and cards with ids, titles, a 200-character `textPreview`, `hidden` and `slideNumber`.
  - Sources: [MCP tools](https://developers.gamma.app/mcp/mcp-tools-reference).
- **Input:** card boundaries can be set in input text with `\n---\n` when `cardSplit: "inputTextBreaks"`. `textMode` is `generate`, `condense` or `preserve` ([Generate API parameters](https://developers.gamma.app/guides/generate-api-parameters-explained)). Gamma 5 drops both and takes a free `prompt` ([migration guide](https://developers.gamma.app/get-started/migrate-from-classic-to-gamma-5)).
- **Data model:** a gamma has **pages** (webpage or multi-page sites), and pages contain **cards** ([MCP tools](https://developers.gamma.app/mcp/mcp-tools-reference)).

## Implications for SiNiHu

These are facts and options only. The decision belongs to the grilling ticket "The Card model: flowing, fixed canvas or hybrid".

- **Gamma has itself moved from flowing to a hybrid.**
  - Classic cards were fluid by default with an optional minimum ratio.
  - Gamma 5 makes fixed 16:9 the presentation canvas, scales content down to fit, and keeps fluid as a per-document option.
  - Gamma 5 adds freeform absolute boxes over a reflowing grid.
  - This is evidence that a pure-flow model was hard to export faithfully. It is also evidence that the "both" option is viable.
- **Options observed:**
  - **(a) Flowing only.** Simplest editor. Export of tall cards is undefined: Gamma documents no page-splitting rule.
  - **(b) Fixed canvas with auto-shrink to a minimum size, then a visible overflow warning.** Export matches the screen. Overflow is resolved by hand, by splitting the slide.
  - **(c) Per-document mode (fluid vs fixed).** Gamma 5 does this.
  - **(d) Grid flow plus opt-in absolute "freeform" elements.** Gamma 5 makes this one-way: freeform cannot go back to the grid.
- **Export fidelity drives the model.** Gamma ties "export matches screen" to fixed slides only, and Zoom to fit does nothing for fluid. Its stated PPTX gaps (gradients, frosted effects, fonts, page backgrounds, blank master) show which styles SiNiHu may want to constrain if PPTX matters.
- **Present mode for tall content:** Gamma scrolls with Up/Down. The alternative, scale-to-fit, is what fixed mode does. SiNiHu could pick either per card mode.
- **AI scope:** Gamma offers deck, card and selected-element scopes. Edits are atomic per request with a before and after diff, and the API can target card IDs. That suggests stable card and element IDs in the document model.
- **Representation:** Gamma exposes no structured document JSON publicly; it offers only text plus an outline with card IDs. SiNiHu is free to choose its own schema. There is no external format to stay compatible with, apart from `\n---\n`-style card breaks in text input as a convenience.
