# Provider Connections and Secrets

Type: grilling
Status: open
Blocked by: 01

## Question

Define the user-facing lifecycle for connecting, validating, replacing, refreshing, and deleting GitHub Copilot access, Azure-hosted BYOK models, and generic OpenAI-compatible BYOK endpoints. Specify which provider settings and secrets are required, how authentication expiry or revocation is handled, and how the UI communicates connection state without ever returning stored secret values. SiNiHu stores secrets server-side and does not include them in project archives.