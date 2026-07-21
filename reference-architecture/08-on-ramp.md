# On-ramp — what to build first

← [Back to Table of Contents](./README.md)

Don't try to stand up the whole stack at once. Sequence it so each step
produces a reusable artifact.

1. **LiteLLM + one MCP server (Google Workspace).** Prove model-agnostic
   routing and self-hosted tool access. One Compose service, one MCP server,
   any model. This is the kernel.
2. **One Inngest function on top — intake or follow-up.** Scheduled,
   retryable, calls a model via LiteLLM, reads/writes via MCP. This is your
   first transferable ops pattern — and it's code, not a visual flow, so it's
   an asset you can version, eval, and hand to a client as-is. (If you need a
   same-day throwaway prototype instead, n8n is the faster tool for *that* —
   just don't let it become the durable pattern.)
3. **Evals and observability for that flow.** Seed promptfoo golden cases
   (5a), wire Langfuse tracing (5b), and establish the closed loop (5c) from
   day one — not as a later add-on. The flow is now defensible, not vibes.
4. **One LangGraph agent — a research pipeline.** Reuses the same LiteLLM and
   MCP layer. This is the agent-tier reference.
5. **Package as the handoff artifact.** Compose file + README + eval harness,
   versioned and tested per 6d — including the rotation and backup runbooks
   (3b, 4f) and the escalation signals (6e). This is the artifact you show
   prospects.
