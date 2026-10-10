# Canonical editable deck model and package

The manual editor, Deck AI, Slide AI, online presenter, and PPTX exporter need one shared editable source rather than renderer-specific documents. Use a versioned deck manifest with ordered typed slide elements, immutable per-slide snapshots, and full-deck checkpoints that reference those snapshots; package retained history and local assets in a self-contained ZIP. This keeps edits and recovery interoperable across surfaces while allowing external video/embed URLs without placing credentials or executable embed code in project archives.
