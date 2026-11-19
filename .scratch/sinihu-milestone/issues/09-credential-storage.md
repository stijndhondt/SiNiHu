# Credential storage for self-hosted

Type: grilling
Status: open
Blocked by: 07

## Question

How are BYOK credentials stored and used in a self-hosted instance? Cover:

- where the master key comes from (environment, file, KMS later)
- encryption at rest
- rotation and deletion
- making sure keys never reach the browser
- how SDK or subscription logins are stored or proxied, if [AI integration approach and provider abstraction](07-ai-integration-approach.md) keeps that lane

The design should stay tenant-aware for hosting later.
