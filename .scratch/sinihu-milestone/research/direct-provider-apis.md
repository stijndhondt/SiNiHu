# Direct provider APIs and abstraction libraries

Research for ticket [02-direct-provider-apis](../issues/02-direct-provider-apis.md). Researched 2026-11-19 against official docs, official SDK repos and the npm registry. Every claim cites its source. Claims marked **UNVERIFIED** could not be confirmed in a primary source during this pass.

Scope: OpenAI, Anthropic, Google Gemini, OpenRouter, xAI, Mistral and custom OpenAI-compatible endpoints, called from a NestJS backend with user-stored (BYOK) keys. SiNiHu needs a structured JSON deck, so structured outputs and partial-JSON streaming matter most.

## Summary

- All six named providers offer JSON-schema-constrained output, but **schema dialects differ**. OpenAI requires every field in `required` and `additionalProperties: false`. Anthropic rejects recursive schemas, `oneOf` and numeric/length constraints. Gemini ignores some keywords without raising an error. xAI treats some keywords as best-effort.
- All of them stream over SSE. Structured output streams as **partial JSON text** that the client must accumulate and parse leniently. No provider streams a validated partial object.
- **Image generation** exists on OpenAI, Gemini (Nano Banana), xAI, OpenRouter, and Mistral (Mistral only as a built-in tool). Anthropic has no image generation.
- Every provider has an authenticated **model-listing** endpoint except OpenRouter, whose `/api/v1/models` is public. The metadata is uneven. Anthropic, Gemini, OpenRouter and xAI return limits or capabilities. OpenAI returns only `id`/`created`/`owned_by`.
- **Key validation:** the cheapest check is a `GET` on the models list, or a key-info endpoint (`/v1/api-key` on xAI, `/api/v1/key` on OpenRouter). None of these generates tokens.
- **Abstractions:** the Vercel AI SDK (Apache-2.0, v7) covers every target provider through first-party or provider-maintained packages and includes a provider registry. TanStack AI (MIT, v1.7) and LangChain.js (MIT, v1.6) are viable but cover fewer targets (TanStack) or are heavier (LangChain).

## 1. Capability tables

### 1.1 Structured outputs (JSON schema)

| Provider | How to request | Notable schema limits | Source |
|---|---|---|---|
| OpenAI | Responses: `text.format = {type:"json_schema", strict:true, schema}`. Chat Completions: `response_format` json_schema | Root must be an object (no root `anyOf`). Every field in `required`, so optional fields use a `null` union. `additionalProperties:false` on every object. ≤5,000 properties, ≤10 nesting levels, ≤1,000 enum values. `allOf`/`not`/`if`/`then`/`else` unsupported. `$ref`/`$defs` and recursion supported. Available from GPT-4o onward | [structured-outputs guide](https://developers.openai.com/api/docs/guides/structured-outputs) |
| Anthropic | `output_config.format = {type:"json_schema", schema}` on Messages. Status stable, no beta header | Constrained decoding. `additionalProperties:false` required. No recursive schemas, `oneOf`, `not`, numeric (`minimum`/`maximum`) or string-length constraints. Unsupported keywords return 400. Grammar compiles on first use and is cached for 24 h. Changing the schema invalidates the prompt cache | [structured-outputs](https://platform.claude.com/docs/en/build-with-claude/structured-outputs) |
| Gemini | Current docs (Interactions API): `response_format = {type:"text", mime_type:"application/json", schema}`. Legacy `generateContent` uses `generationConfig.responseMimeType` + `responseJsonSchema` | Supports types, `properties`/`required`/`additionalProperties`, `enum`, `format`, `minimum`/`maximum`, `items`/`prefixItems`/`minItems`/`maxItems`. **Silently ignores** `minLength`, `maxLength`, `minProperties`, `maxProperties`, `uniqueItems` and `contains` | [structured-output](https://aistudio.google.com/docs/structured-output), [generate-content ref](https://aistudio.google.com/api-reference/generate-content) |
| OpenRouter | Chat `response_format` json_schema or Responses `text.format`. Use `provider.require_parameters: true` to route only to endpoints that support it | Support depends on the model and the upstream endpoint. "exact compliance is not guaranteed on every endpoint" | [structured-outputs](https://openrouter.ai/docs/features/structured-outputs) |
| xAI | `response_format = {type:"json_schema", ...}` (OpenAI-compatible) | `anyOf`/`oneOf`, one `allOf` and non-circular `$ref` supported. `additionalProperties` defaults to false. Limits such as 64 object properties and array size 256 are **best-effort** beyond those numbers. `not` and `if`/`then` are best-effort only | [structured-outputs](https://docs.x.ai/docs/guides/structured-outputs) |
| Mistral | `response_format = {type:"json_schema", ...}` on `/v1/chat/completions`. The Responses API uses `text.format` | Documented as "guarantees" that output follows the schema. Supported schema keywords and the meaning of `strict`: **UNVERIFIED** | [API ref](https://docs.mistral.ai/api), [custom SO](https://docs.mistral.ai/studio-api/conversations/structured-output/custom), [Responses](https://docs.mistral.ai/studio/conversations/responses) |
| Custom OpenAI-compatible | Depends on the server. Clients must treat it as a capability flag. The AI SDK models this as `supportsStructuredOutputs` | Unknown per server | [AI SDK openai-compatible](https://ai-sdk.dev/providers/openai-compatible-providers) |

Gemini's OpenAI-compatibility endpoint also accepts structured output through `chat.completions.parse` ([openai compat](https://aistudio.google.com/docs/openai)).

### 1.2 Streaming and partial structured output

| Provider | Streaming transport | Partial structured output | Source |
|---|---|---|---|
| OpenAI | SSE with semantic events (`response.created`, `response.output_text.delta`, `response.completed`). Chat Completions uses `stream: true` | Structured output streams as text deltas. The docs recommend the SDK streaming helpers. `output_text.done` does not mean the response succeeded, so check the final status and any refusal | [streaming](https://developers.openai.com/api/docs/guides/streaming-responses), [structured-outputs](https://developers.openai.com/api/docs/guides/structured-outputs) |
| Anthropic | SSE (`message_start`, `content_block_delta` with `text_delta` / `input_json_delta`, `message_delta`, `message_stop`) | Works with streaming, but "you need to accumulate the full response before deserializing". Tool inputs stream as `partial_json` | [streaming](https://platform.claude.com/docs/en/build-with-claude/streaming), [structured-outputs](https://platform.claude.com/docs/en/build-with-claude/structured-outputs) |
| Gemini | `streamGenerateContent` (legacy) and `stream=True` on Interactions | "The streamed chunks are valid partial JSON strings that can be concatenated to form the final JSON object" | [structured-output](https://aistudio.google.com/docs/structured-output), [generate-content ref](https://aistudio.google.com/api-reference/generate-content) |
| OpenRouter | `stream: true` (SSE) | "streams partial JSON that forms a valid, schema-matching response once complete". Response Healing is available only for non-streaming requests | [structured-outputs](https://openrouter.ai/docs/features/structured-outputs) |
| xAI | SSE data-only events ending in `data: [DONE]` | "The chunks will progressively build up the JSON string" | [responses ref](https://docs.x.ai/docs/api-reference), [structured-outputs](https://docs.x.ai/docs/guides/structured-outputs) |
| Mistral | `stream: true`. Data-only SSE ending in `data: [DONE]` | Streaming combined with `json_schema` is not documented: **UNVERIFIED** | [API ref](https://docs.mistral.ai/api) |

On every provider the backend receives a growing JSON string and must parse it leniently to render slides progressively.

### 1.3 Image generation

| Provider | Image generation | Source |
|---|---|---|
| OpenAI | `POST /v1/images/generations` with `gpt-image-*` models (returns `b64_json`), or the Responses `image_generation` tool. `partial_images` 0–3 streams partial images | [image-generation](https://developers.openai.com/api/docs/guides/image-generation) |
| Anthropic | None. Not offered by the Messages API (absence: no image-generation feature in the Claude docs) | **UNVERIFIED** (absence claim) |
| Gemini | Nano Banana models via `generate_content` (images arrive as content parts). Imagen is retired. The OpenAI-compat endpoint supports `images.generate` with `gemini-2.5-flash-image` / `gemini-3-pro-image-preview` | [image-generation](https://ai.google.dev/gemini-api/docs/image-generation), [openai compat](https://aistudio.google.com/docs/openai) |
| OpenRouter | Dedicated `POST /api/v1/images` (returns `b64_json` or a URL that expires after 24 h). Image models are listed at `/api/v1/images/models` or `/api/v1/models?output_modalities=image` | [image-generation](https://openrouter.ai/docs/features/multimodal/image-generation) |
| xAI | `POST /v1/images/generations` and `/v1/images/edits`. `response_format` is `url` or `b64_json`. Deferred mode polls `GET /v1/images/{request_id}` | [images ref](https://docs.x.ai/developers/rest-api-reference/inference/images) |
| Mistral | No standalone endpoint. A built-in `image_generation` tool exists in Agents/Conversations and (per the tools overview) Chat Completions. Docs disagree on whether it returns a file ID or a URL | [image_generation tool](https://docs.mistral.ai/studio-api/agents/agent-tools/image_generation), [agent tools](https://docs.mistral.ai/studio-api/agents/agent-tools) |
| Custom OpenAI-compatible | Only if the server implements `/images/generations` | — |

### 1.4 Model listing

| Provider | Endpoint | Useful metadata | Source |
|---|---|---|---|
| OpenAI | `GET /v1/models` | `id`, `created`, `owned_by`, `shutdown_date`. **No** capabilities, context window or pricing | [models list](https://developers.openai.com/api/reference/resources/models/methods/list) |
| Anthropic | `GET /v1/models` (cursor pagination, `limit` ≤1000, `lifecycle` filter) | `capabilities` (incl. `structured_outputs`, `image_input`, `thinking`, `effort`), `max_input_tokens`, `max_tokens`, `lifecycle`, `display_name` | [models list](https://platform.claude.com/docs/en/api/models/list) |
| Gemini | `GET /v1beta/models` | `inputTokenLimit`, `outputTokenLimit`, `supportedGenerationMethods`, `displayName` | [models ref](https://aistudio.google.com/api-reference/models) |
| OpenRouter | `GET /api/v1/models` (public, filterable by `supported_parameters` and `output_modalities`). `GET /api/v1/models/user` needs auth | `pricing`, `context_length`, `supported_parameters` (e.g. `structured_outputs`) | [list models](https://openrouter.ai/docs/api-reference/list-available-models) |
| xAI | `GET /v1/models`, `/v1/language-models`, `/v1/image-generation-models` (+ `/{id}`) | Pricing, modalities, aliases | [models ref](https://docs.x.ai/developers/rest-api-reference/inference/models) |
| Mistral | `GET /v1/models`, `/v1/models/{id}` | `capabilities` (`completion_chat`, `function_calling`, `vision`, …), `max_context_length`, `aliases`. The docs give inconsistent response shapes (bare array vs `data`) | [models endpoint](https://docs.mistral.ai/api/endpoint/models) |
| Custom OpenAI-compatible | `GET {baseURL}/models` if implemented | Usually `id` only (**UNVERIFIED**: depends on the server) | — |

### 1.5 Usage / token reporting

| Provider | Non-streaming | Streaming | Source |
|---|---|---|---|
| OpenAI | Chat: `usage.prompt_tokens`, `completion_tokens`, `total_tokens`, plus `*_details` | Chat: `stream_options.include_usage` adds a chunk before `[DONE]`. Responses API usage fields: **UNVERIFIED** in this pass | [chat create](https://developers.openai.com/api/reference/resources/chat/subresources/completions/methods/create) |
| Anthropic | `usage` on the Message | `message_start` carries usage. `message_delta.usage` counts are **cumulative**. Separate free `count_tokens` endpoint | [streaming](https://platform.claude.com/docs/en/build-with-claude/streaming), [token counting](https://platform.claude.com/docs/en/build-with-claude/token-counting) |
| Gemini | `usageMetadata`: `promptTokenCount`, `candidatesTokenCount`, `thoughtsTokenCount`, `cachedContentTokenCount`, `totalTokenCount`, … | Same object on stream chunks (**UNVERIFIED** which chunk) | [generate-content ref](https://aistudio.google.com/api-reference/generate-content) |
| OpenRouter | Always included, with no opt-in. Includes `cost` (USD charged) and `cost_details` | In the terminal event | [usage accounting](https://openrouter.ai/docs/use-cases/usage-accounting) |
| xAI | `input_tokens`, `output_tokens`, `total_tokens`, cached/reasoning details, **`cost_in_usd_ticks`** (10^10 ticks per USD) | Not described | [responses ref](https://docs.x.ai/docs/api-reference) |
| Mistral | Required `usage` object (`UsageInfo`). Subfields not shown in the reference (**UNVERIFIED**, presumably OpenAI-style) | **UNVERIFIED** | [API ref](https://docs.mistral.ai/api) |
| Custom OpenAI-compatible | Server-dependent. The AI SDK needs `includeUsage: true` to request streamed usage | — | [AI SDK openai-compatible](https://ai-sdk.dev/providers/openai-compatible-providers) |

Only OpenRouter and xAI report **cost** directly. For the other providers, SiNiHu would compute cost from tokens using its own price table.

### 1.6 Cheapest key validation ("test connection")

| Provider | Cheapest documented check | Notes | Source |
|---|---|---|---|
| OpenAI | `GET /v1/models` | No tokens generated. That it is free is **UNVERIFIED** (not stated explicitly) | [models list](https://developers.openai.com/api/reference/resources/models/methods/list) |
| Anthropic | `GET /v1/models`. Alternative: `POST /v1/messages/count_tokens` | count_tokens is documented as "**free to use**" with separate RPM limits. A models GET also returns per-model capabilities | [models list](https://platform.claude.com/docs/en/api/models/list), [token counting](https://platform.claude.com/docs/en/build-with-claude/token-counting) |
| Gemini | `GET /v1beta/models` with `x-goog-api-key` | Free: **UNVERIFIED** | [models ref](https://aistudio.google.com/api-reference/models) |
| OpenRouter | `GET /api/v1/key` | Returns `limit`, `limit_remaining`, usage totals and `is_free_tier`. Do **not** use `/api/v1/models` for this, because it is public and does not validate the key. Subject to per-IP burst limits | [key/limits](https://openrouter.ai/docs/api-reference/limits) |
| xAI | `GET /v1/api-key` | Returns `api_key_blocked`, `api_key_disabled`, `team_blocked` and `acls`. Keys can have zero permissions by default ("all requests fail if this field is empty"), so check the ACLs as well | [account ref](https://docs.x.ai/developers/rest-api-reference/inference/other), [management guide](https://docs.x.ai/developers/management-api-guide) |
| Mistral | `GET /v1/models` | Free: **UNVERIFIED** | [models endpoint](https://docs.mistral.ai/api/endpoint/models) |
| Custom OpenAI-compatible | `GET {baseURL}/models`. Fall back to a `max_tokens: 1` chat call | Not all servers implement `/models` (**UNVERIFIED**) | — |

## 2. Abstraction libraries vs a thin in-house layer

### 2.1 Candidates

| | Vercel AI SDK (`ai`) | TanStack AI (`@tanstack/ai`) | LangChain.js (`langchain`) | Official vendor SDKs (in-house layer) |
|---|---|---|---|---|
| Version / license | 7.0.177, Apache-2.0 ([npm](https://registry.npmjs.org/ai/latest), [LICENSE](https://raw.githubusercontent.com/vercel/ai/main/LICENSE)) | 1.7.0, MIT ([npm](https://registry.npmjs.org/@tanstack/ai/latest), [repo](https://github.com/TanStack/ai)) | 1.6.7, MIT ([npm](https://registry.npmjs.org/langchain/latest)) | `openai` 7.60.0, `@anthropic-ai/sdk` 1.11.0, `@google/genai` 3.6.1, `@mistralai/mistralai` 3.5.0: all Apache-2.0 (npm registry, queried 2026-11-19) |
| Maturity signal | 28.1k stars, ~9.8k commits, major v7 ([repo](https://github.com/vercel/ai)) | 3.4k stars, ~1.3k commits ([repo](https://github.com/TanStack/ai)) | Long-lived, v1 line. Star count not checked | Vendor-maintained |
| OpenAI / Anthropic / Gemini | `@ai-sdk/openai`, `@ai-sdk/anthropic`, `@ai-sdk/google` ([providers](https://ai-sdk.dev/providers/ai-sdk-providers)) | `ai-openai`, `ai-anthropic`, `ai-gemini` ([repo](https://github.com/TanStack/ai)) | `@langchain/openai`, `/anthropic`, `/google` ([providers](https://docs.langchain.com/oss/javascript/integrations/providers/overview)) | One SDK each |
| xAI | `@ai-sdk/xai` | `@tanstack/ai-grok` | `@langchain/xai` | Use the `openai` SDK with a base URL (xAI is OpenAI-compatible) |
| Mistral | `@ai-sdk/mistral` | **No adapter listed** | `@langchain/mistralai` | `@mistralai/mistralai` |
| OpenRouter | `@openrouter/ai-sdk-provider` (OpenRouter-maintained, [page](https://ai-sdk.dev/providers/community-providers/openrouter)) | `@tanstack/ai-openrouter` | `@langchain/openrouter` | `@openrouter/sdk` or the `openai` SDK |
| Custom OpenAI-compatible | `@ai-sdk/openai-compatible`, with flags `supportsStructuredOutputs` and `includeUsage` ([docs](https://ai-sdk.dev/providers/openai-compatible-providers)) | **No generic adapter listed** | UNVERIFIED (likely `@langchain/openai` with `baseURL`) | `openai` SDK with `baseURL` |
| Structured output | `generateText`/`streamText` with `output: Output.object({schema})`. Zod/Valibot/JSON Schema. Validates the final result ([docs](https://ai-sdk.dev/docs/ai-sdk-core/generating-structured-data)) | `outputSchema`. JSON Schema/Zod/ArkType/Valibot ([repo](https://github.com/TanStack/ai)) | `withStructuredOutput`. Agent `responseFormat` with `providerStrategy` (native) or `toolStrategy` (tool-call emulation, the default) ([docs](https://docs.langchain.com/oss/javascript/langchain/structured-output)) | Hand-written per provider. Must translate one schema into each provider's dialect (§1.1) |
| Partial structured streaming | `partialOutputStream`. Partial results are not schema-validated. `elementStream` yields validated array elements ([docs](https://ai-sdk.dev/docs/ai-sdk-core/generating-structured-data)) | UNVERIFIED | Not documented on the structured-output page (UNVERIFIED) | Hand-written partial-JSON parser |
| Image generation | `generateImage` across OpenAI, Google, xAI, Fal, etc. Per-call `usage` ([docs](https://ai-sdk.dev/docs/ai-sdk-core/image-generation)) | Yes for some adapters, e.g. Grok images, fal ([repo](https://github.com/TanStack/ai)) | UNVERIFIED | Per provider |
| Per-task model routing | `createProviderRegistry` (`'provider:model'` string IDs) and `customProvider` (aliases, default settings, restricted model sets) ([docs](https://ai-sdk.dev/docs/ai-sdk-core/provider-management)) | Adapters imported per activity (e.g. `openaiText`) (search summary of [docs](https://tanstack.com/ai/latest/docs), UNVERIFIED detail) | Model objects passed per chain/agent | Your own `task → {provider, model}` map |
| Model listing / key validation | Not part of the abstraction (UNVERIFIED). Needs direct HTTP calls either way | Not covered (UNVERIFIED) | Not covered (UNVERIFIED) | Direct |

### 2.2 Lock-in and other notes

- **AI SDK gateway default:** in the AI SDK, plain model strings route through the Vercel AI Gateway by default ([repo README](https://github.com/vercel/ai)). A BYOK self-hosted app would instantiate providers explicitly, e.g. `createOpenAI({ apiKey })`. This per-user key pattern is not shown on the provider-management page, so it is **UNVERIFIED** there, although `apiKey` is a documented provider option in `createOpenAICompatible` ([docs](https://ai-sdk.dev/providers/openai-compatible-providers)).
- **AI SDK churn:** structured output moved from the older `generateObject`/`streamObject` to `output` on `generateText`/`streamText`. The current docs describe only the latter ([docs](https://ai-sdk.dev/docs/ai-sdk-core/generating-structured-data)). Major versions (now v7) have changed APIs. The size of each migration is UNVERIFIED.
- **Node version:** the AI SDK README states Node.js 22+ ([repo](https://github.com/vercel/ai)).
- **Lock-in surface:** all three libraries are permissively licensed (Apache-2.0/MIT) and run self-hosted, so the lock-in is in the API shape, not in a service. That holds as long as the AI SDK gateway is not used. LangChain's agent/chain abstractions are the heaviest surface. The AI SDK's `LanguageModel` spec is also the extension point for community providers ([providers](https://ai-sdk.dev/providers/ai-sdk-providers)).
- **Gemini API churn:** Google's docs now present an **Interactions API** (`POST /v1beta/interactions`) as primary, and `generateContent` docs are labelled "Legacy" ([structured-output](https://aistudio.google.com/docs/structured-output), [legacy page title](https://ai.google.dev/gemini-api/docs/generate-content/image-generation)). An in-house Gemini adapter would have to track this change. An abstraction absorbs it.
- **OpenAI-compatible shortcut:** xAI, Mistral, OpenRouter and Gemini (through `/v1beta/openai/`) all expose OpenAI-style Chat Completions ([Gemini compat](https://aistudio.google.com/docs/openai), [Mistral API](https://docs.mistral.ai/api), [xAI SO](https://docs.x.ai/docs/guides/structured-outputs), [OpenRouter SO](https://openrouter.ai/docs/features/structured-outputs)). A thin layer could therefore cover most providers with one OpenAI-compatible client plus a native Anthropic client. Gemini's compatibility mode is partial: forum reports describe failures with optional fields in structured output, a non-primary source, so **UNVERIFIED**.

## 3. Implications for SiNiHu

Facts and options only. The decision belongs to the grilling ticket "AI integration approach and provider abstraction" ([07](../issues/07-ai-integration-approach.md)).

**Facts**

1. All six providers support schema-constrained JSON, but with different subsets. A single deck schema must stay within the intersection or be adapted per provider. The intersection roughly excludes recursion (Anthropic), `oneOf`/`allOf`/`not`, numeric and length constraints (Anthropic; ignored by Gemini), and optional fields that are not expressed as `null` unions (OpenAI strict).
2. Progressive slide rendering always requires **lenient partial-JSON parsing on the backend**, whether in-house or via the AI SDK's `partialOutputStream`. Partial objects are never schema-validated, so validation happens on completion, or per array element via the AI SDK's `elementStream`.
3. Anthropic has no image generation, and Mistral offers it only as an agent/chat tool. Image generation therefore needs its own task routing, independent of the text provider.
4. Model metadata is uneven. Anthropic, Gemini, OpenRouter, xAI and Mistral expose limits or capabilities, and OpenAI exposes only IDs. A "pick a model" UI needs a local capability table for at least OpenAI and custom endpoints.
5. Key validation can be token-free for every named provider. OpenRouter and xAI need their key-info endpoints instead of the models endpoint: OpenRouter's models endpoint is public, and xAI keys can lack ACLs.
6. Cost is reported natively only by OpenRouter and xAI. The others need a price table.

**Options**

- **A. Vercel AI SDK:** covers all 7 targets (including `openai-compatible` and the OpenRouter-maintained provider), has a provider registry for per-task routing, plus `generateImage` and partial structured streaming. It is Apache-2.0. Costs: v-major churn, the gateway default must be avoided, and model listing and key validation are still hand-written.
- **B. TanStack AI:** MIT and a younger project. Neither a Mistral adapter nor a generic OpenAI-compatible adapter is listed, so two of the seven targets would need custom adapters.
- **C. LangChain.js:** MIT and covers all named providers. Its structured output defaults to tool-call emulation unless `providerStrategy` is chosen. It is the heaviest abstraction for a single-call JSON generation use case.
- **D. Thin in-house layer:** one OpenAI-compatible client covers OpenAI, xAI, Mistral, OpenRouter, custom endpoints and Gemini compat, plus a native Anthropic client and optionally a native Gemini client. SiNiHu would own schema-dialect translation, partial-JSON parsing, usage normalisation and Gemini API churn.
- **E. Hybrid:** use an abstraction for generation calls (text, object, image), plus a small in-house module for listing models, validating keys and holding a capability/price table. That module is needed under every option.
