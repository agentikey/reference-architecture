# Design principles

← [Back to Table of Contents](./README.md)

1. **No single vendor owns the workflow.** Models are pluggable; orchestration
   and tooling are owned, not rented. A vendor can be swapped without rewriting
   the IP.
2. **Self-hosted MCP over cloud connectors.** The data path is auditable. The
   same MCP server runs for Agentikey and for a client; only the credentials
   and target data change.
3. **Two orchestration tiers, not one.** Business ops (scheduled, retryable,
   stateful) and agent work (multi-step reasoning, research) have different
   shapes. Don't force a single framework to do both.
4. **Eat your own dog food.** Every flow built for Agentikey is a battle-tested
   pattern transferable to a client. Internal tooling = reference architecture.
5. **Patterns over syntax.** Bet on durable skills (planning, state, tools,
   evals). Treat any specific framework as replaceable.
