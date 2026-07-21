# Agentikey Reference Architecture — Vendor-Neutral AI Operations

> Draft v0.1. The named stack Agentikey runs on *and* the pattern installed for
> clients. Iterative — refine as real engagements surface constraints.

A reference architecture for a vendor-neutral AI operations stack — the named
tools Agentikey runs on internally and the pattern installed for clients.
Each layer makes a specific, defensible choice and names the trade-off
honestly. The whole is designed so that nothing a client runs depends on a
single AI vendor.

## The stack

| # | Section | Summary |
|---|---------|---------|
| 0 | [Design principles](./00-design-principles.md) | The five load-bearing principles — no vendor owns the workflow, self-hosted MCP over cloud connectors, two orchestration concerns, eat your own dog food, patterns over syntax. |
| 1 | [Model access](./01-model-access.md) | Three shapes for the model layer: no gateway, self-hosted gateway (on-thesis default), or reseller (off-thesis). Why gateway value reframed around operations, not interface unification. |
| 2 | [Orchestration](./02-orchestration.md) | Python-native split by concern: Inngest/Temporal for durable execution, LangGraph for reasoning. Why n8n is demoted from spine to situational tool. |
| 3 | [Tools (MCP)](./03-tools-mcp.md) | Self-hosted MCP servers as owned assets. Selection rubric, auth (the hard part), least-privilege scoping, transport, custom servers as IP, and glue to Layer 2. |
| 4 | [Persistence](./04-persistence.md) | SQLite for local-first, Postgres as the convergence engine, vector strategy, embeddings as a sticky decision, residency as selling point, portability/backup/migration. |
| 5 | [Evals & observability](./05-evals-observability.md) | Offline evals vs online observability as distinct layers. The closed loop (golden cases + traces → CI gating → failures become cases) as the actual moat. |
| 6 | [Deployment](./06-deployment.md) | Docker Compose on a $10-30/mo VPS. Service inventory, host choice, networking/TLS/secrets, the Compose file as handoff artifact, K8s escalation, maintenance cadence. |

## Supporting sections

| Section | Summary |
|---------|---------|
| [The boundary](./07-boundary.md) | What's reusable IP (yours, amortized across engagements) vs client-specific (replaced per engagement). |
| [On-ramp](./08-on-ramp.md) | Five-step sequencing where each step produces a reusable artifact, not just learning. |
| [Honest trade-offs](./09-trade-offs.md) | The cost of the thesis — setup tax, maintenance, framework volatility, MCP's youth — reframed as the business model. |
| [What this is not](./10-what-this-is-not.md) | Not a Claude replacement, not anti-vendor, not a productized platform, not a finished answer. |

## How to read this

- **For a prospect:** skim the summaries above, then read
  [Design principles](./00-design-principles.md) and
  [Deployment](./06-deployment.md) for the concrete pitch ($10-30/mo,
  no vendor dependency).
- **For a build:** start at the [On-ramp](./08-on-ramp.md) and follow the
  sequence — each step yields a reusable artifact.
- **For a client audit:** [Model access](./01-model-access.md),
  [Tools](./03-tools-mcp.md), and [Evals](./05-evals-observability.md) are
  where vendor dependencies and defensibility live.
