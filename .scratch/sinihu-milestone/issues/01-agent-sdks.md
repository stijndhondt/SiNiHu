# Agent SDKs: Copilot, Claude Agent and Codex

Type: research
Status: resolved
Blocked by:

## Question

What can GitHub Copilot SDK (especially its BYOK mode, which may cover providers that have no SDK of their own), Claude Agent SDK and Codex SDK do for SiNiHu? For each SDK, cover:

- auth modes (API key, subscription or login, BYOK)
- runtime needs (bundled or spawned CLI process? language and runtime?)
- whether it runs server-side in a self-hosted NestJS backend, and later in multi-user hosting
- structured-output (JSON schema) support and streaming
- model and provider coverage
- image generation
- usage reporting
- license and terms on using a user's subscription login inside a third-party app

Docs:
- https://docs.github.com/en/copilot/how-tos/copilot-sdk/setup/choosing-a-setup-path
- https://code.claude.com/docs/en/agent-sdk/overview
- https://learn.chatgpt.com/docs/codex-sdk

## Answer

Full findings: [../research/agent-sdks.md](../research/agent-sdks.md)

- All three SDKs are agent runtimes, not thin model clients. Each has a Node/TS package and spawns a CLI process: the Copilot CLI over JSON-RPC (bundled), the `claude` binary (bundled, one subprocess per session) and the `codex` CLI (JSONL over stdio). Each can run in a NestJS backend at the cost of a process per session plus local disk state.
- Copilot SDK (MIT, being renamed "Microsoft Copilot SDK") BYOK needs no GitHub auth. It supports OpenAI, Azure, Anthropic, Google Gemini (experimental) and any OpenAI-compatible endpoint (Ollama, vLLM, LiteLLM), so it can act as a multi-provider layer. Structured output (`responseSchema`) is in preview and depends on the provider.
- Subscription login: Copilot officially supports GitHub OAuth, billed to each user's Copilot plan. Anthropic forbids claude.ai login in third-party products without approval. OpenAI says Codex app-server ChatGPT auth was never permitted for commercial or hosted services; Sign in with ChatGPT (preview) covers OSS / local apps, and hosted apps go through an interest form.
- Structured output: Claude has validated JSON Schema with retry, Codex has `outputSchema` per turn, Copilot's is in preview. All three stream.
- Usage: Copilot reports `assistant.usage` per call, Claude reports tokens plus an estimated `total_cost_usd`, Codex reports `turn.completed.usage` tokens. None of these is authoritative billing data.
- Image generation: Claude has none. Copilot has an experimental native image tool behind Copilot auth. Codex has `$imagegen` (`gpt-image-2`) in the CLI/app, but SDK exposure is unverified.
