# What this stack deliberately is not

← [Back to Table of Contents](./README.md)

- Not a Claude replacement. Claude stays a default model for hard reasoning,
  accessed through LiteLLM. You keep the model, lose the lock-in.
- Not anti-vendor. It's pro-portability. A client on OpenAI-only is fine; the
  point is the architecture doesn't *force* that.
- Not a productized platform to resell as-is. The *architecture* is a
  reference pattern you adapt per engagement, not a shrink-wrapped product.
  That said, the *services around it* are explicitly productized — eval
  harness installation (5e), the cost-and-reliability audit (5e), maintenance
  retainers (6f). The distinction: sell the work of installing and maintaining
  the pattern, not the pattern itself as a sealed artifact.
- Not a finished answer. This doc is v0.1, pressure-tested layer by layer but
  not yet battle-tested against a real engagement. Expect revisions when real
  clients surface constraints the draft didn't anticipate — particularly in
  MCP auth (3b) and the agent-framework choice (Layer 2), both of which are
  still-standardizing spaces.
