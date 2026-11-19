# AI integration approach and provider abstraction

Type: grilling
Status: open
Blocked by: 01, 02

## Question

Does SiNiHu integrate AI through the agent-SDK lane, the direct-API lane, or both?

What shape does the provider abstraction take? Cover what it exposes to the generation pipeline (structured generation, streaming, image generation, usage events), how per-task model routing (text, reasoning, image) is configured, and how a provider or connection is added.

Inputs: [Agent SDKs: Copilot, Claude Agent and Codex](01-agent-sdks.md), [Direct provider APIs and abstraction libraries](02-direct-provider-apis.md).
