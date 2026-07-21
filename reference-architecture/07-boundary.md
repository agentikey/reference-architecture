# The boundary — reusable IP vs client-specific

← [Back to Table of Contents](./README.md)

This boundary is what makes it a *reference architecture* and not a one-off.

**Reusable IP (yours, amortized across engagements):**
- The Compose deployment template (versioned, tested — see 6d)
- LiteLLM routing config + fallback rules + cost-tracking setup
- Inngest/Temporal workflow patterns (intake, follow-up, invoicing, sync)
- LangGraph agent skeletons (research, plan→act) + their checkpoint configs
- The eval harness, golden-case *format*, and judge-calibration protocol
- The Langfuse/observability config and dashboard templates
- MCP server selection, hardening playbook, and rotation runbooks
- Custom MCP servers built for common client systems (see 3e)

**Client-specific (replaced per engagement):**
- MCP server credentials and target data
- Prompts tuned to their domain
- Model choice per their budget / residency / policy
- Persistence choice (SQLite vs Postgres) per their scale
- Deployment target (their cloud vs managed VPS)

The pitch writes itself: *"We install the orchestrator once and swap the
client-specific layer. Nothing you run depends on a single AI vendor."*
