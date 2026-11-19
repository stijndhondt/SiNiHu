# Direct provider APIs and abstraction libraries

Type: research
Status: resolved
Blocked by:

## Question

For calling OpenAI, Anthropic, Gemini, OpenRouter, xAI, Mistral and custom OpenAI-compatible endpoints directly from a NestJS backend with stored keys:

- What does each support for structured outputs, streaming, image generation, model listing and usage/token reporting?
- How cheaply can a key be validated (the "test connection" feature)?
- Should SiNiHu adopt an existing TypeScript abstraction (e.g. Vercel AI SDK, or others found) or build a thin layer of its own? Compare coverage, maturity and lock-in.

## Answer

Findings: [../research/direct-provider-apis.md](../research/direct-provider-apis.md)

- All six providers support JSON-schema-constrained output, but their schema dialects differ (for example OpenAI strict requires every field, and Anthropic forbids recursion, `oneOf` and numeric constraints). A deck schema must target the common subset or be adapted per provider.
- All of them stream over SSE. Structured output arrives as partial JSON text, so the backend needs lenient partial parsing. No provider streams a validated partial object.
- OpenAI, Gemini (Nano Banana), xAI and OpenRouter offer image generation, and Mistral offers it only as a tool. Anthropic has none, so image generation needs separate routing.
- A token-free key check exists for every provider: `GET /models`, or `/api/v1/key` (OpenRouter, whose models list is public) and `/v1/api-key` (xAI). Only OpenRouter and xAI report cost, so the others need a price table.
- The Vercel AI SDK (Apache-2.0, v7) covers all seven targets and has a provider registry for per-task routing. TanStack AI (MIT) lists no Mistral or generic OpenAI-compatible adapter. LangChain.js (MIT) covers all of them but is heavier. A thin layer (one OpenAI-compatible client plus native Anthropic) is feasible. Model listing and key validation are hand-written under every option. The choice is deferred to ticket 07.
