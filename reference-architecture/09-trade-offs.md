# Honest trade-offs (the cost of the thesis)

← [Back to Table of Contents](./README.md)

These are restated here not because the layers above didn't cover them, but
because they share a single theme: **the vendor-neutral thesis has a
standing cost, and the consultancy's economics depend on that cost being
worth paying.** Each trade-off below is the price of an asset in the layers
above. The engagement model only works if clients can't justify paying this
price for one stack but gladly pay you to amortize it across many.

- **Setup tax.** More moving parts than Claude Desktop connectors or a
  managed agent platform. Paid for by IP that amortizes across engagements
  (Layer 2's honest tax; 6d's versioned handoff). This is the gap clients
  hire you to close — don't undersell it.
- **Ongoing maintenance.** MCP servers, LiteLLM, Inngest/Temporal, Langfuse
  all upgrade; credentials rotate; eval golden sets need growth; backups need
  restore drills. The full cadence is in 6f — and that cadence is itself the
  retainer that keeps the relationship alive past handoff.
- **Framework volatility in the agent tier.** LangGraph is the safe bet today;
  the agent-framework space churns. The hedge is patterns-over-syntax (Layer
  2) and containerization that makes the orchestrator swappable (6e) without a
  rewrite.
- **MCP is converging but young.** Bet on the standard; stay skeptical of
  any single server's long-term maintenance. The selection rubric (3a) and
  pre-install audit (3c) are how you operationalize that skepticism — and,
  honestly, how you bill for staying ahead of a moving spec.

The unifying framing: **these aren't bugs in the architecture, they're the
business model.** Each trade-off is work clients would otherwise have to do
themselves or do without. The consultancy's job is to make that work
recurring, billable, and visibly worth it.
