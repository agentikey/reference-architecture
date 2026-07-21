# Layer 2 — Orchestration (Python-native, two concerns)

← [Back to Table of Contents](./README.md)

The orchestration layer splits on *concern*, not language. Both tiers are
Python — one durable-execution framework for scheduled/retryable stateful
work, one agent framework for branching reasoning. They compose rather than
compete: a durable workflow orchestrates the overall job and invokes a
LangGraph agent inside a step where that job needs reasoning. One language,
two concerns, both owned as code.

**Why Python-native is the on-thesis default (and why n8n isn't, despite
its popularity).** Two facts reshaped this choice:

1. *n8n is no longer open source.* It's source-available under a Sustainable
   Use License ("fair-code"), not OSI-approved. Internal self-hosted use is
   permitted, but hosting n8n as a service or productizing n8n-based
   solutions without a paid license is restricted. Every client you install
   it on inherits a license dependency that's less clean than an Apache/MIT
   Python service — a yellow flag for a vendor-neutral consultancy, the same
   quiet re-lock-in pattern the architecture is supposed to prevent.
2. *MCP dissolves n8n's main advantage.* The reason to reach for n8n was
   "hundreds of pre-built integrations so you don't write auth and
   integration code." But if MCP is the tool layer, that integration code is
   already written — it lives in the MCP servers, which are reusable assets
   you own. Your orchestrator just calls MCP tools. n8n's integration library
   becomes redundant the moment MCP is the standard. **MCP + Python means you
   own both the tool layer and the orchestration layer, and the integrations
   are assets, not configurations of someone else's product.**

A third reason seals it for delivery: *code is more defensible IP than
visual flows.* Python beats n8n's exported JSON for a consultancy producing
evaluable, versionable, reviewable deliverables. You can run promptfoo
against code, diff it in git, hire anyone to maintain it. An n8n workflow
handed to a client is a configuration of a product they now depend on; a
Python service handed to a client is an asset they own outright. That
difference *is* the vendor-neutral pitch.

---

**Tier A — Durable execution:** **Inngest** (Python SDK) as default
- Why: code-first, event-driven, "built for how agents actually fail."
  Built-in retries, scheduling, queues, human-in-the-loop, state that
  survives restarts. Lighter weight than Temporal and faster to stand up —
  the right default for most small-AI-company engagements.
- Pairs naturally with LangGraph: an Inngest function can invoke a LangGraph
  agent as one durable step in a larger workflow.

**Tier A upgrade — Temporal:** for clients who need serious durability
- Multi-day agentic workflows, full replay history, enterprise-grade
  reliability and observability. Heavier to operate but the boring, hardened
  choice when an engagement has real uptime and audit requirements.
- Note: the OpenAI Agents SDK pairs with Temporal — a sign this combination
  is converging, which lowers the bet risk.

**Tier B — Agent / multi-step reasoning:** **LangGraph**
- Why: explicit state graphs, real maintenance, broadest adoption, least
  likely to disappear. The boring, durable bet for branching reasoning and
  tool loops — research pipelines, client-facing agents, anything with
  non-trivial decision flow.
- Rising alternative: **PydanticAI** — pick this instead if starting fresh
  and you value type-safety and a cleaner mental model over maturity. Don't
  run both. Pick one, learn it deep.
- Treat as replaceable. The durable skill is the plan→tools→state→eval
  pattern, not LangGraph's API.

---

**Where n8n still fits.** Demoted from spine to a situational tool, kept as
a legitimate client option when the tradeoffs are accepted eyes-open:

- *Personal velocity / throwaway glue.* For Agentikey's own quick prototype
  flows where reuse isn't the goal, n8n's visual editor is faster than
  writing Python. Fine to use — just don't make it the architecture's spine.
- *Clients who explicitly accept the tradeoff.* Some clients want a visual
  workflow tool their non-technical staff can maintain after handoff, and are
  willing to accept the Sustainable Use License and the product dependency in
  exchange. That's a legitimate client choice — the job is to make it an
  *informed* one. When you install n8n for a client, document the license
  constraint and flag it as a known dependency in the audit, the same way you'd
  flag an OpenRouter dependency in the model layer. The principle isn't
  "never n8n"; it's "never n8n by default, and never n8n undocumented."

The honest tax: Python orchestration is more initial code than clicking nodes.
It pays off because MCP already absorbed the integration work, the code is
reusable client IP, and you're not installing a license-encumbered dependency
everywhere. For a one-off glue job with no reuse intent, n8n is the faster
tool for that job — just don't let one-off convenience become the spine.
