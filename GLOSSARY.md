# SiNiHu Decks

Terms for the editable presentation model shared by deck editing, AI authoring, online presentation, and export.

## Language

**Deck**:
An editable presentation with a defined slide size and an ordered collection of slides.
_Avoid_: Presentation file, when referring to the editable source model

**Slide**:
A single canvas in a deck, with its own content and revision history.
_Avoid_: Page

**Element**:
A typed, positioned item on a slide, such as text, an image, a shape, a table, a chart, or an external resource.
_Avoid_: Object, widget

**Slide revision**:
An immutable snapshot of one slide's complete content at a committed edit boundary.
_Avoid_: Change, delta

**Deck checkpoint**:
A snapshot of a deck's ordered slide-to-revision mapping.
_Avoid_: Deck revision, save point

**Deck package**:
A versioned ZIP archive containing a deck manifest, retained history, and local assets referenced by that history.
_Avoid_: Export, PPTX

**External resource**:
A video or provider embed represented by a credential-free HTTPS URL and metadata, rather than embedded HTML or scripts.
_Avoid_: Embed code
