# Copilot Session Boundary

Type: grilling
Status: open
Blocked by: 01, 05

## Question

Specify how SiNiHu authorizes and routes AI-session operations between an authenticated account, its project, and the shared backend Copilot runtime. Define ownership checks for create, resume, use, and delete; session identifiers and persistence; behavior when provider credentials expire; and serialization for multiple tabs acting on the same user's session. Use the agreed shared runtime with `mode: "empty"`, explicit capabilities, and project-scoped session retention; do not rely on SDK session IDs as authorization.