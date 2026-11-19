# Agent SDKs: Copilot, Claude Agent and Codex

Research for ticket `../issues/01-agent-sdks.md`. Gathered 2026-11-19 from primary sources only (official docs, official repos, terms). Every claim has a source link. Anything not confirmed in a primary source is marked **UNVERIFIED**.

## TL;DR

- All three SDKs wrap an **agent runtime that runs as a separate CLI or runtime process**: the Copilot CLI over JSON-RPC, the `claude` binary over stdio, and the `codex` CLI over JSONL on stdio. All three have a TypeScript package that runs in Node, so they can sit inside a NestJS backend. Each concurrent session costs a subprocess, plus local disk state.
- **Copilot SDK BYOK** is the only one of the three that is genuinely multi-provider. It supports native `openai`, `azure`, `anthropic` and `google` (experimental) provider types, plus any OpenAI-compatible endpoint (Ollama, vLLM, LiteLLM, Foundry Local). With BYOK, no GitHub auth and no Copilot subscription is required.
- **Subscription login in a third-party app:**
  - Copilot explicitly documents GitHub OAuth so each user's own Copilot subscription is billed.
  - Anthropic forbids offering claude.ai login in third-party products unless pre-approved.
  - OpenAI says Codex app-server ChatGPT auth "has never been permitted for commercial or hosted services". It points to "Sign in with ChatGPT" instead, which is in preview and covers open-source and locally hosted apps; paid or remotely hosted apps must use an interest form.

## 1. GitHub Copilot SDK (being renamed "Microsoft Copilot SDK")

### Identity, license, status
- The SDK is MIT licensed. The repo `LICENSE` reads "MIT License, Copyright GitHub, Inc." ([repo](https://github.com/github/copilot-sdk), [LICENSE](https://github.com/github/copilot-sdk/blob/main/LICENSE)).
- The README says the project is GA and follows semver, and is moving from the "GitHub Copilot SDK" name to "Microsoft Copilot SDK" ([README](https://github.com/github/copilot-sdk)).
- The Node package in the README is now `@microsoft/copilot-sdk` ([nodejs/README.md](https://github.com/github/copilot-sdk/blob/main/nodejs/README.md)). docs.github.com still shows `npm install @github/copilot-sdk` ([choosing a setup path](https://docs.github.com/en/copilot/how-tos/copilot-sdk/setup/choosing-a-setup-path)).
- Official SDKs: Node/TS, Python, Go, .NET, Rust and Java ([README](https://github.com/github/copilot-sdk)).

### Runtime
- "Every Copilot SDK integration follows the same core pattern": app → SDK → Copilot CLI over JSON-RPC ([choosing a setup path](https://docs.github.com/en/copilot/how-tos/copilot-sdk/setup/choosing-a-setup-path)).
- **Bundled CLI by default.** The Node, Python and .NET SDKs include the CLI; Go needs a separate install ([same page](https://docs.github.com/en/copilot/how-tos/copilot-sdk/setup/choosing-a-setup-path)).
- **Node packaging.** The CLI ships in an optional `@microsoft/copilot-sdk-<platform>` package, and "starting the SDK performs no runtime download". `COPILOT_CLI_PATH` overrides it ([nodejs README](https://github.com/github/copilot-sdk/blob/main/nodejs/README.md)).
- **Node version:** Node `^20.19.0 || >=22.12.0` ([nodejs README](https://github.com/github/copilot-sdk/blob/main/nodejs/README.md)).
- **Connection modes:** spawn over stdio, spawn as a TCP server, or connect to an already-running runtime ([nodejs README](https://github.com/github/copilot-sdk/blob/main/nodejs/README.md)).
- **In-process runtime (experimental):** runs without a separate CLI process ([choosing a setup path](https://docs.github.com/en/copilot/how-tos/copilot-sdk/setup/choosing-a-setup-path)).
- **Backend services:** run `copilot --headless --port 4321`, and the SDK connects over TCP, so the "CLI and app can run in different containers" ([backend services](https://docs.github.com/en/copilot/how-tos/copilot-sdk/setup/backend-services)).
  - "There is no official pre-built Docker image for the Copilot CLI."
  - Session state is on local disk.
  - "No built-in auth between SDK and CLI."

### Auth modes
- **GitHub OAuth (user's own subscription).** Users authorize your GitHub App or OAuth App, and you pass their token ([GitHub OAuth setup](https://docs.github.com/en/copilot/how-tos/copilot-sdk/setup/github-oauth)).
  - "Each user needs an active Copilot subscription."
  - "Copilot usage is billed to each user's subscription."
  - Accepted token types: `gho_`, `ghu_` and `github_pat_`. Classic `ghp_` tokens are not accepted.
- **Service or server-to-server token.** A GitHub App installation token (`ghs_`) bills an organization; no signed-in user is required ([server-to-server tokens](https://github.com/github/copilot-sdk/blob/main/docs/auth/server-to-server-tokens.md)).
- **BYOK.** BYOK "bypass[es] GitHub Copilot authentication" ([BYOK](https://docs.github.com/en/copilot/how-tos/copilot-sdk/auth/byok)).
  - The README FAQ says a Copilot subscription is required "unless you are using BYOK", and "BYOK - Use your own API keys (no GitHub auth required)" ([README](https://github.com/github/copilot-sdk)).
  - "BYOK uses key-based authentication only". Entra ID, managed identities and third-party IdPs are not supported per the README FAQ ([README](https://github.com/github/copilot-sdk)). This contradicts a separate docs page on Azure Managed Identity with BYOK ([choosing a setup path](https://docs.github.com/en/copilot/how-tos/copilot-sdk/setup/choosing-a-setup-path)); the README FAQ may be stale.
  - Bearer-token and callback-token options exist (`bearerToken`, `bearerTokenProvider`, `apiKeyCommand`) ([BYOK](https://docs.github.com/en/copilot/how-tos/copilot-sdk/auth/byok)).

### BYOK provider and model coverage (the key question)

Supported provider types ([repo docs/auth/byok.md](https://github.com/github/copilot-sdk/blob/main/docs/auth/byok.md); docs.github.com lists the same set minus `google`, see [BYOK](https://docs.github.com/en/copilot/how-tos/copilot-sdk/auth/byok)):

| Provider | `type` |
|---|---|
| OpenAI and OpenAI-compatible | `openai` |
| Microsoft Foundry / Azure OpenAI | `openai` or `azure` |
| Anthropic (Claude) | `anthropic` (always uses the Messages API) |
| Google Gemini | `google` (**experimental**, native Gemini Interactions API) |
| Ollama, Foundry Local, vLLM, LiteLLM, other OpenAI-compatible servers | `openai` |

- `wireApi` selects `completions` or `responses` for OpenAI/Azure ([byok.md](https://github.com/github/copilot-sdk/blob/main/docs/auth/byok.md)).
- `model` is required with a custom provider. The CLI may not know the provider's model list, so you can supply `onListModels` ([BYOK](https://docs.github.com/en/copilot/how-tos/copilot-sdk/auth/byok)).
- **Through an OpenAI-compatible endpoint, any provider with such an endpoint can be used,** for example Mistral, OpenRouter or a LiteLLM proxy. Whether a specific provider works end to end (tool calls, structured output) is **UNVERIFIED** per provider.
- BYOK limitations ([byok.md](https://github.com/github/copilot-sdk/blob/main/docs/auth/byok.md)):
  - only your provider's models are available;
  - your provider's rate limits apply;
  - "Usage is tracked by your provider, not GitHub Copilot";
  - requests do not count against premium request quotas.
- Without BYOK, "All models available via Copilot CLI are supported in the SDK" ([README](https://github.com/github/copilot-sdk)).

### Server-side and multi-user
- Multi-user servers must use `mode: "empty"` ("Required for multi-user or shared scenarios") and set a `gitHubToken` per session. Session state lives under `COPILOT_HOME/session-state/{sessionId}`, and the docs warn against sharing one service token across users ([multi-tenancy](https://docs.github.com/en/copilot/how-tos/copilot-sdk/setup/multi-tenancy)).
- Scaling guidance exists ([scaling](https://docs.github.com/en/copilot/how-tos/copilot-sdk/setup/scaling)).
- Per-session BYOK configuration in a multi-tenant server: BYOK is configured per `createSession` (`inference.providers`) ([nodejs README](https://github.com/github/copilot-sdk/blob/main/nodejs/README.md)). The multi-tenancy page does not describe it, so per-user BYOK keys on one shared runtime are **UNVERIFIED** as an officially documented pattern.

### Structured output and streaming
- **Structured output (preview).** Pass a JSON Schema or Zod schema as `responseSchema`. The typed `sendAndWait(prompt, schema)` parses and validates the result ([nodejs README](https://github.com/github/copilot-sdk/blob/main/nodejs/README.md)).
  - It "Requires a runtime build with `responseFormat` … support".
  - "Provider schema restrictions still apply."
  - "An API-compatible gateway may ignore format fields even when it accepts the request."
- **Streaming.** With streaming on, `assistant.message_delta` events carry `deltaContent`. The final `assistant.message` event is always sent ([nodejs README](https://github.com/github/copilot-sdk/blob/main/nodejs/README.md); [streaming events](https://github.com/github/copilot-sdk/blob/main/docs/features/streaming-events.md)).

### Image generation
- There is an **experimental native image tool**. `ProtocolImageGenerationOptions` is described as "Opt into native CAPI image generation", is "Disabled by default for SDK sessions", and "CAPI model access and permissions still apply". `ImageToolModel` supports `generate` and `edit` ([nodejs/src/generated/rpc.ts](https://github.com/github/copilot-sdk/blob/main/nodejs/src/generated/rpc.ts)).
- CAPI is the Copilot API, so this depends on GitHub auth. Image generation through **BYOK** providers is **UNVERIFIED** (not documented).
- Image **input** is supported as PNG, JPEG, WebP and GIF attachments ([nodejs README](https://github.com/github/copilot-sdk/blob/main/nodejs/README.md)).

### Usage reporting
- The `assistant.usage` event fires per model call with `model`, `inputTokens`, `outputTokens`, cache tokens and `cost` (the premium-request multiplier) ([usage-and-billing](https://github.com/github/copilot-sdk/blob/main/docs/features/usage-and-billing.md)).
- `session.usage.getMetrics` returns accumulated totals and is experimental ([same page](https://github.com/github/copilot-sdk/blob/main/docs/features/usage-and-billing.md)).
- `account.getQuota` returns account quota ([same page](https://github.com/github/copilot-sdk/blob/main/docs/features/usage-and-billing.md)).
- With BYOK, provider-side tracking is authoritative ([byok.md](https://github.com/github/copilot-sdk/blob/main/docs/auth/byok.md)).

### Terms on using a user's subscription
- GitHub documents this flow officially: users sign in with GitHub and their own subscription is billed ([GitHub OAuth setup](https://docs.github.com/en/copilot/how-tos/copilot-sdk/setup/github-oauth)).
- Usage is billed "based on the same model as the Copilot CLI" ([README](https://github.com/github/copilot-sdk)).
- **UNVERIFIED:** I found no SDK-specific clause in GitHub's contractual terms (GitHub Terms of Service or Copilot product terms) that addresses third-party apps. I found no prohibition in primary sources either.

## 2. Claude Agent SDK

### Identity, license
- Python and TypeScript only. "A library that runs the Claude Code binary" ([overview](https://code.claude.com/docs/en/agent-sdk/overview)).
- **Not open source.** Use is "governed by Anthropic's Commercial Terms of Service, including when you use it to power products and services that you make available to your own customers and end users" ([overview](https://code.claude.com/docs/en/agent-sdk/overview); [Commercial Terms](https://www.anthropic.com/legal/commercial-terms)).
- Branding rules: "Powered by Claude" is allowed. "Claude Code" naming is not ([overview](https://code.claude.com/docs/en/agent-sdk/overview)).

### Runtime
- The SDK bundles a native Claude Code binary, and the TypeScript package installs it via npm optional dependencies. It needs Node 18.14+ or Python 3.10+ ([quickstart](https://code.claude.com/docs/en/agent-sdk/quickstart)).
- "The Agent SDK spawns and supervises a `claude` CLI subprocess that owns a shell, a working directory, and session files on disk." One session maps to one subprocess ([hosting](https://code.claude.com/docs/en/agent-sdk/hosting)).
- Suggested starting resources: 1 GiB RAM, 5 GiB disk and 1 CPU per agent ([hosting](https://code.claude.com/docs/en/agent-sdk/hosting)).

### Auth modes
- Auth options ([quickstart](https://code.claude.com/docs/en/agent-sdk/quickstart)):
  - `ANTHROPIC_API_KEY`;
  - Amazon Bedrock (`CLAUDE_CODE_USE_BEDROCK=1`);
  - Claude Platform on AWS;
  - Google Cloud's Agent Platform, formerly Vertex (`CLAUDE_CODE_USE_VERTEX=1`);
  - Microsoft Foundry (`CLAUDE_CODE_USE_FOUNDRY=1`).
- `ANTHROPIC_BASE_URL` can route model calls through a proxy ([hosting](https://code.claude.com/docs/en/agent-sdk/hosting)).
- **Subscription login:** "Unless previously approved, Anthropic does not allow third party developers to offer claude.ai login or rate limits for their products, including agents built on the Claude Agent SDK. Use the API key authentication methods … instead." ([overview](https://code.claude.com/docs/en/agent-sdk/overview), [quickstart](https://code.claude.com/docs/en/agent-sdk/quickstart)).

### Server-side and multi-user
- Self-hosting patterns are documented: ephemeral, long-running, hybrid with a `SessionStore`, and multi-agent containers ([hosting](https://code.claude.com/docs/en/agent-sdk/hosting)).
- Multi-tenant isolation ([hosting](https://code.claude.com/docs/en/agent-sdk/hosting)):
  - `settingSources: []`;
  - `CLAUDE_CODE_DISABLE_AUTO_MEMORY=1`;
  - per-tenant `CLAUDE_CONFIG_DIR` and `cwd`.
- **Per-user keys:** the API key is read from the subprocess environment, and `env` can be passed per `query()` ([hosting](https://code.claude.com/docs/en/agent-sdk/hosting)). That makes a per-request user key mechanically possible, but no docs page names that pattern, so it is **UNVERIFIED** as a documented pattern.

### Structured output and streaming
- **Structured output.** Set `outputFormat: { type: "json_schema", schema }`. The result message carries a validated `structured_output`, and the SDK re-prompts when the output does not match ([structured outputs](https://code.claude.com/docs/en/agent-sdk/structured-outputs)).
  - JSON Schema draft-07; Zod with `target: "draft-7"`.
  - Error subtype `error_max_structured_output_retries`.
- **Streaming.** `includePartialMessages: true` yields raw API `stream_event`s such as `content_block_delta`/`text_delta` ([streaming output](https://code.claude.com/docs/en/agent-sdk/streaming-output)).
  - With structured output on, the JSON streams as unvalidated `input_json_delta` chunks.
  - Only the final result is validated ([same page](https://code.claude.com/docs/en/agent-sdk/streaming-output)).

### Model and provider coverage
- Claude models only, served by Anthropic, Bedrock, Google Cloud, Foundry or Claude Platform on AWS ([quickstart](https://code.claude.com/docs/en/agent-sdk/quickstart)).
- Anthropic "doesn't support routing Claude Code to non-Claude models through any gateway" ([LLM gateway](https://code.claude.com/docs/en/llm-gateway)).

### Image generation
- None. "Claude is an image understanding model only. It can interpret and analyze images, but it cannot generate, produce, edit, manipulate, or create images." ([vision](https://platform.claude.com/docs/en/build-with-claude/vision)).

### Usage reporting
- The result message carries `total_cost_usd`, `usage` and per-model `modelUsage` with tokens, cache tokens and cost ([cost tracking](https://code.claude.com/docs/en/agent-sdk/cost-tracking)).
- Per-step usage appears on assistant messages ([same page](https://code.claude.com/docs/en/agent-sdk/cost-tracking)).
- Cost is a "client-side estimate". The docs say "Don't bill end users … from these fields" ([same page](https://code.claude.com/docs/en/agent-sdk/cost-tracking)).
- OpenTelemetry export is available ([hosting](https://code.claude.com/docs/en/agent-sdk/hosting)).

## 3. Codex SDK (OpenAI)

### Identity, license
- TypeScript `@openai/codex-sdk` (Node 18+) and Python `openai-codex` (3.10+) ([Codex SDK](https://learn.chatgpt.com/docs/codex-sdk)).
- The docs say "Use the library server-side" ([codex-sdk.md](https://learn.chatgpt.com/docs/codex-sdk.md)).
- Repo license: Apache-2.0 ([openai/codex](https://github.com/openai/codex), [LICENSE](https://github.com/openai/codex/blob/main/LICENSE)).

### Runtime
- TS SDK: "It spawns the CLI and exchanges JSONL events over stdin/stdout" ([sdk/typescript README](https://github.com/openai/codex/tree/main/sdk/typescript)).
- Python SDK: "controls the local Codex app-server over JSON-RPC", with a pinned Codex CLI runtime dependency ([Codex SDK](https://learn.chatgpt.com/docs/codex-sdk)).
- Threads persist in `~/.codex/sessions` ([sdk/typescript README](https://github.com/openai/codex/tree/main/sdk/typescript)).
- "Codex requires the working directory to be a Git repository" unless you pass `skipGitRepoCheck` ([sdk/typescript README](https://github.com/openai/codex/tree/main/sdk/typescript)).
- For richer custom clients, the **app-server** speaks JSON-RPC over stdio. Its WebSocket transport "is experimental and unsupported" ([app-server](https://learn.chatgpt.com/docs/app-server)).

### Auth modes
- "Sign in with ChatGPT for subscription access" or "Sign in with an API key for usage-based access". "API keys are still the recommended default for automation." ([auth](https://learn.chatgpt.com/docs/auth)).
- The TS SDK's `CodexOptions` has `apiKey`, `baseUrl`, `env` and `config` ([codexOptions.ts](https://github.com/openai/codex/blob/main/sdk/typescript/src/codexOptions.ts)). The SDK injects `CODEX_API_KEY` ([README](https://github.com/openai/codex/tree/main/sdk/typescript)).
- App-server login types: `apiKey`, `chatgpt`, `chatgptDeviceCode` and experimental `chatgptAuthTokens`. The last is "intended for host apps that already own the user's ChatGPT auth lifecycle." Amazon Bedrock is also an auth mode ([app-server](https://learn.chatgpt.com/docs/app-server)).

### Terms on using a user's ChatGPT subscription
- "App-server authentication has never been permitted for commercial or hosted services." Existing local or open-source apps may continue, and OpenAI recommends migrating to **Sign in with ChatGPT** ([app-server](https://learn.chatgpt.com/docs/app-server)).
- **Sign in with ChatGPT (SIWC)** lets users "use their ChatGPT plan for eligible AI requests in your app" and requires requesting a client ID ([SIWC](https://developers.openai.com/siwc)).
- The plan-usage add-on is documented for "open-source and locally hosted apps". For "a paid or remotely hosted app, complete the interest form" ([SIWC token sharing](https://developers.openai.com/siwc/token-sharing-open-source)).
- SIWC preview limits ([preview limitations](https://developers.openai.com/siwc/token-sharing-open-source/preview-limitations); [models and inference](https://developers.openai.com/siwc/token-sharing-open-source/models-and-inference)):
  - Responses API only, with `store:false` and `stream:true`.
  - Many parameters are unsupported, including `temperature` and `max_output_tokens`.
  - **Image generation is unsupported.**
  - The app must refresh the OAuth token.

### Server-side and multi-user
- The SDK is meant for server-side use ([codex-sdk.md](https://learn.chatgpt.com/docs/codex-sdk.md)), but each thread is a local CLI process with on-disk state ([README](https://github.com/openai/codex/tree/main/sdk/typescript)).
- Multi-user hosting is fine with API keys. With ChatGPT login, hosted use is not permitted via app-server auth ([app-server](https://learn.chatgpt.com/docs/app-server)). SIWC for remotely hosted apps is gated behind an interest form ([SIWC token sharing](https://developers.openai.com/siwc/token-sharing-open-source)).
- No official multi-tenant isolation guide was found: **UNVERIFIED**.

### Structured output and streaming
- `thread.run(prompt, { outputSchema })` produces JSON that conforms to the schema; Zod works via `zodToJsonSchema(..., { target: "openAi" })` ([README](https://github.com/openai/codex/tree/main/sdk/typescript)).
- On the app-server, "`outputSchema` applies only to the current turn" ([app-server](https://learn.chatgpt.com/docs/app-server)).
- `runStreamed()` returns an async generator of events such as `item.completed` and `turn.completed` ([README](https://github.com/openai/codex/tree/main/sdk/typescript)).
- Token-level text deltas in the TS SDK event set are **UNVERIFIED**; the README shows item-level events.

### Model and provider coverage
- OpenAI models by default. Available models "depend on the client and account" ([app-server](https://learn.chatgpt.com/docs/app-server)).
- Custom `model_providers.<id>` entries take `base_url`, `env_key` and `wire_api`. The docs' example includes Mistral and local Ollama ([advanced config](https://learn.chatgpt.com/docs/config-file/config-advanced.md)).
- Built-in providers: `openai`, `ollama`, `lmstudio`, `amazon-bedrock`, `amazon-bedrock-runtime` and `gcp` (Vertex) ([advanced config](https://learn.chatgpt.com/docs/config-file/config-advanced.md)).
- The SDK can pass this config through its `config` option ([README](https://github.com/openai/codex/tree/main/sdk/typescript)).
- How well non-OpenAI models perform inside Codex's agent loop is **UNVERIFIED**.

### Image generation
- Codex has built-in image generation via the `$imagegen` skill, which uses `gpt-image-2` and counts toward Codex usage limits. Setting `OPENAI_API_KEY` routes generation to API pricing ([image generation](https://learn.chatgpt.com/docs/image-generation)).
- The docs cover the app, the IDE extension and the CLI, not the SDK. Availability through the SDK is **UNVERIFIED**.
- Image **input**: `local_image` entries are passed to the CLI via `--image` ([README](https://github.com/openai/codex/tree/main/sdk/typescript)).

### Usage reporting
- `turn.completed.usage` carries `input_tokens`, `cached_input_tokens`, `cache_write_input_tokens` and `output_tokens` ([events.ts](https://github.com/openai/codex/blob/main/sdk/typescript/src/events.ts)).
- The app-server adds `thread/tokenUsage/updated` and `account/rateLimits/read` ([app-server](https://learn.chatgpt.com/docs/app-server)).
- No cost field was found in the SDK: **UNVERIFIED**.

## Comparison table

| | Copilot SDK | Claude Agent SDK | Codex SDK |
|---|---|---|---|
| License | MIT | Anthropic Commercial Terms (proprietary) | Apache-2.0 |
| TS package / Node | `@microsoft/copilot-sdk` (docs: `@github/copilot-sdk`), Node ≥20.19 | `@anthropic-ai/claude-agent-sdk`, Node ≥18.14 | `@openai/codex-sdk`, Node ≥18 |
| Runtime | Bundled Copilot CLI over JSON-RPC (stdio, TCP or headless server); in-process mode experimental | Spawns bundled `claude` binary, one subprocess per session | Spawns `codex` CLI, JSONL over stdio; Git repo cwd by default |
| API key auth | BYOK (no GitHub auth needed) | `ANTHROPIC_API_KEY`, Bedrock, Google Cloud, Foundry | `apiKey` / `CODEX_API_KEY`; Bedrock, Vertex |
| User subscription login in your app | **Documented and supported** (GitHub OAuth; each user needs Copilot and is billed) | **Not allowed** unless pre-approved | **Not permitted** for commercial or hosted use via app-server; SIWC preview for OSS / local apps, interest form for hosted |
| Providers | Copilot catalog, or BYOK: OpenAI, Azure, Anthropic, Google (exp.), any OpenAI-compatible endpoint | Claude only | OpenAI, plus configurable OpenAI-compatible providers (Ollama, LM Studio, Mistral example), Bedrock, Vertex |
| Structured output | `responseSchema` (preview; provider-dependent) | `outputFormat` JSON Schema, validated with retry | `outputSchema` per turn |
| Streaming | `assistant.message_delta` | `includePartialMessages` raw deltas | `runStreamed()` item/turn events |
| Image generation | Experimental native CAPI image tool (needs Copilot auth); BYOK UNVERIFIED | None | `$imagegen` (`gpt-image-2`) in CLI/app; via SDK UNVERIFIED |
| Usage reporting | `assistant.usage` per call; metrics and quota RPCs | `total_cost_usd` (estimate), `usage`, `modelUsage` | `turn.completed.usage` tokens; no cost |
| Multi-tenant guidance | Yes (`mode:"empty"`, per-session token) | Yes (isolation options, `SessionStore`) | None found (UNVERIFIED) |

## Implications for SiNiHu

Facts and options only. The decision belongs to the grilling ticket "AI integration approach and provider abstraction" (`../issues/07-ai-integration-approach.md`).

### Facts
- All three SDKs are **agent runtimes**, not thin model clients. Each needs a CLI process per session, local disk state, and tool and permission configuration. SiNiHu's core need, schema-shaped JSON for slides, may only need a single structured model call.
- All three have Node/TS packages, so each can run inside a NestJS backend. The cost is a child process (or TCP runtime) per session. Claude's docs suggest about 1 GiB of RAM per agent as a starting point.
- **Subscription login (Gamma-style "use my plan")**:
  - only Copilot officially supports it in a third-party server app (GitHub OAuth, billed to each user's Copilot plan);
  - Claude forbids it without approval;
  - OpenAI forbids it via app-server for commercial or hosted use. SIWC (preview) covers OSS / locally hosted apps; hosted use needs an interest form.
- **Copilot BYOK can act as a multi-provider abstraction**: OpenAI, Azure, Anthropic, Gemini (experimental), Ollama and any OpenAI-compatible endpoint, with no GitHub account needed. Structured output then depends on each provider's native support.
- **Image generation:**
  - Copilot has an experimental native tool, behind Copilot auth;
  - Codex has `$imagegen`, but SDK exposure is unverified and SIWC excludes image generation;
  - Claude has none.
- **Usage reporting:**
  - Copilot reports per-call tokens;
  - Claude reports tokens plus an *estimated* cost;
  - Codex reports per-turn tokens.
  - None are authoritative for billing end users.

### Options the grilling ticket can weigh
1. **Copilot SDK as the single integration layer.** GitHub OAuth for users with Copilot, plus BYOK for everyone else and other providers. Trade-offs: a CLI runtime dependency, structured output still in preview, and the SDK rename / package churn.
2. **Per-vendor agent SDKs** (Claude Agent SDK, Codex SDK) behind SiNiHu's own provider interface. In practice this means API keys only, given the login restrictions.
3. **Direct provider APIs** (see ticket 02) for generation, optionally adding an agent SDK only where agentic behaviour (tools, multi-step) is needed.
4. **Hybrid.** Direct APIs or BYOK as the default, plus Copilot OAuth as a "use my subscription" option. The Codex SIWC preview is a possible future option if SiNiHu is open source and locally hosted, or gets approval for hosted use.
