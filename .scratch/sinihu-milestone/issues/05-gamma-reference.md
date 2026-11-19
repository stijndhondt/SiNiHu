# Gamma as reference behaviour

Type: research
Status: resolved
Blocked by:

## Question

How does Gamma.app's card model behave? Cover flowing vs fixed height, how layouts and elements are positioned, how it handles overflow, how Cards present full-screen, and how it maps Cards to PPTX/PDF on export, including known fidelity complaints.

Use Gamma's own docs and help center as primary sources. This is reference behaviour only: SiNiHu builds its own architecture and look.

## Answer

Full findings with citations: [../research/gamma-reference.md](../research/gamma-reference.md)

- **Height model:** Classic Gamma cards were fluid by default, with an optional minimum ratio. Gamma 5 makes fluid vs fixed a per-gamma setting. Fixed 16:9 slides scale content down to a minimum readable size, and past that a grey overflow stripe appears. Fixing overflow is manual: split the slide or switch to fluid.
- **Layout:** Classic used columns, Smart Layouts and accent-image layouts. Gamma 5 replaces these with reflowing grids plus opt-in absolute "freeform" boxes, and a box cannot go back from freeform to grid.
- **Present mode:** Up/Down scrolls tall slides and Left/Right moves between slides. Exports render Present mode.
- **Export:** PDF, PPTX, PNG (one image per card) and Google Slides. Only fixed slides are promised to match the screen, and Zoom to fit does nothing for fluid slides. Gamma documents fidelity gaps: gradients, frosted effects, font weights and italics, page backgrounds, a blank slide master, and failures around 150 or more slides. Tables are editable in PPTX. Page splitting of tall fluid cards and general text editability in PPTX are UNVERIFIED.
- **AI and API:** AI editing scopes are the whole deck, one card, or selected elements, and each request is accepted or rejected as a whole. The public REST and MCP API exposes no structured document JSON, only a formatted-text read and an outline with card IDs.
