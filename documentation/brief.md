# Project Brief: OpenDeck — AI Visual Content Creation Platform
## Product vision

Build an AI-powered visual content creation platform inspired by Gamma.app, but designed around an open, Bring Your Own Key (BYOK) architecture.

The platform should allow users to transform ideas, prompts, documents, and existing presentations into beautifully designed, editable visual content.

The objective is to combine the ease of AI-powered content generation with the flexibility of a professional visual editor.

Unlike traditional AI SaaS platforms, the application must not depend on bundled AI credits or mandatory AI subscriptions. Users bring their own API keys and connect directly to their preferred AI providers.

## Core principles
1. BYOK first: Users own their AI provider accounts, API keys, model choices, and AI consumption.
2. Provider agnostic: Support multiple AI providers through a unified integration layer.
3. No mandatory AI subscription: AI usage is billed directly by the user's chosen provider.
4. Editable output: AI-generated content must remain fully editable.
5. Design quality: Prioritize polished visual output, consistent themes, layouts, and typography.
6. AI-assisted workflows: Users can generate, refine, redesign, and restructure content conversationally.
7. Portability: Projects should be exportable and stored in a structured, reusable format.
## Core features
### 1. Dashboard
- Project overview.
- Create new presentation, document, or website.
- Project folders and organization.
- Recent projects.
- Templates and themes.
### 2. AI generation
- Generate content from prompts.
- Generate from pasted text or outlines.
- Import existing documents.
- Generate and refine outlines.
- Automatically create layouts and visual hierarchy.
- Generate relevant images and illustrations.
- Conversational editing.
### 3. Visual editor
- Card-based presentation editor.
- Drag-and-drop editing.
- Rich text and media elements.
- Charts, tables, shapes, and icons.
- Theme customization.
- Manual positioning and resizing.
- AI editing at card and document level.
- Undo, redo, and autosave.
### 4. BYOK configuration
- Secure API key management.
- OpenAI integration.
- Anthropic integration.
- Google Gemini integration.
- OpenRouter integration.
- Extensible provider architecture.
- Model selection per task.
- API connection testing.
- Usage and cost tracking.
- Encrypted server-side credential storage.
### 5. Content formats
- Presentations.
- Pitch decks.
- Reports and documents.
- Social media carousels.
- Infographics.
- Landing pages and websites.
### 6. Import and export
- PDF import.
- PowerPoint import.
- Word document import.
- PDF export.
- PowerPoint export.
- PNG export.
- Shareable web links.
### 7. Design system
- Reusable themes.
- Typography presets.
- Color palettes.
- Layout library.
- Brand kits.
- Reusable content blocks.
- Responsive rendering.

## Technical architecture

Build the application using modular architecture.

Separate:

- Frontend and visual editor.
- Document data model.
- Rendering engine.
- AI provider integrations.
- Prompt and generation orchestration.
- Project and asset management.
- Import and export services.

Use a structured JSON document representation as the source of truth for generated content.

The AI should generate structured content and layout instructions rather than arbitrary frontend code.

The rendering engine should transform the document structure into editable visual elements.

Individual cards should be independently editable and regeneratable.

## Suggested technology stack
Angular + Optimus UI + Tailwind + Angular CDK + Tiptap + NestJS + PostgreSQL is a strong foundation for this product..

## Business model

The platform should not charge users for AI credits.

Users pay their own AI providers directly through their API accounts.

The core editor and essential creation features should be free, with optional monetization for:

- Cloud hosting.
- Additional storage.
- Team collaboration.
- Enterprise workspaces.
- Custom domains.
- Managed infrastructure.

## Initial development milestone

Build a working end-to-end experience:

1. User creates an account.
1. User configures an AI provider and API key.
1. User selects an AI model.
1. User describes a presentation.
1. AI generates an outline.
1. AI generates the complete presentation.
1. User edits individual cards manually or through AI chat.
1. User changes themes and layouts.
1. User exports the result to PDF or PowerPoint.

The final product should feel like a polished commercial-grade AI design application, not a basic chatbot that generates slides.

The user experience and quality of the visual editor are as important as the AI integration itself.