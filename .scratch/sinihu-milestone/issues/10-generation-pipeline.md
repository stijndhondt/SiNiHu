# The generation pipeline

Type: grilling
Status: open
Blocked by: 07, 08

## Question

How does Prompt → Outline → Deck work end to end? Cover:

- whether the user edits the Outline
- how Cards stream into the editor
- background jobs (BullMQ) vs request/response
- regenerating a single Card
- the AI-edit protocol for Card-level and deck-level instructions (JSON patch vs replace)
- when image generation happens
- where token usage is recorded

Inputs: [AI integration approach and provider abstraction](07-ai-integration-approach.md), [The document model schema](08-document-model-schema.md).
