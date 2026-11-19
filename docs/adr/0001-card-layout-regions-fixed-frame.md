# Cards use Layout Regions and a fixed Frame, not an absolute canvas or pure flow

A Card has a **Frame**. The first milestone ships only fixed 16:9, and the model already allows fluid and other ratios. Content is placed through a **Layout** of named **Regions**, and Elements flow in order inside each Region.

Users can pull an Element out as a **Free Element** at a position relative to the Frame. It remembers its home Region and can snap back. Regions do not nest; structured content is a **Compound Element**.

We chose this over a PowerPoint-style absolute canvas because the AI and Themes then work with Layouts and Regions instead of coordinates. That keeps design consistent and makes changing a Layout or Theme, or regenerating a Card, safe.

We chose it over Gamma-Classic-style pure flow because a fixed Frame gives Present mode and PDF/PPTX export a defined result. Gamma itself moved from flow to a fixed 16:9 canvas.

## Consequences

- **Editable PPTX needs a measurement pass.** The render service (headless Chromium, using the editor's Card renderer) measures Element boxes, and PptxGenJS writes them. The same service prints the PDF.
- **Overflow is surfaced, never silently fixed.** Text shrinks automatically down to the Theme's minimum size, then the Card shows an overflow warning (Split, Switch Layout or Shorten). Generation is told to stay within each Region's capacity.
- **Charts are images in the first PPTX export.** They are rendered as images to keep the in-app look; native PPTX charts come later.
- **Free Elements can go back to their Region.** Unlike Gamma's one-way freeform, the move is reversible.

## Considered options

- **Absolute canvas:** simplest PPTX export, but poor AI layout quality and destructive re-theming.
- **Pure flow:** simplest editor, but no defined export or Present behaviour for tall Cards.
- **Grid with one-way freeform (Gamma 5):** rejected because Free Elements should be able to return to their Region.
