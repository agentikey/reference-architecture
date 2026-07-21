# Layer 3 — Tools (self-hosted MCP servers)

← [Back to Table of Contents](./README.md)

All self-hosted, all portable. The portability story for clients: *the
orchestrator stays the same; only the MCP server set and credentials change.*
This layer is where MCP absorbs the integration work that would otherwise
live inside a vendor's product (n8n's node library, Claude Desktop's
connectors) — making the tools reusable assets you own rather than
configurations of someone else's product. It's also where the consultancy
value is densest: auth, scoping, and hardening are the parts clients can't
justify building for one stack but will pay to have installed correctly.

## 3a — Catalog and selection

| MCP server | Source | Covers |
|---|---|---|
| Google Workspace | `j3k0/mcp-google-workspace` | Gmail, Calendar, Drive, Docs |
| GitHub | official `@modelcontextprotocol/server-github` | repos, issues, PRs |
| Filesystem | official | local files ( scoped ) |
| Slack / Notion / Linear | community, per engagement | as client needs dictate |

**Selection rubric** — "prefer official or well-maintained community" is too
vague to be defensible. Clients will ask how you decide which community server
to trust with their credentials. Apply, in order:

1. **Maintenance recency.** Last commit within ~90 days; responsive maintainer
   to issues. A stale MCP server with live credentials is a liability.
2. **Auth method.** OAuth 2.1 / OIDC over static long-lived API keys. If a
   server only supports a static key with no scoping, it's a stopgap, not a
   default.
3. **License.** Apache 2.0 or MIT preferred. Avoid servers with non-commercial
   or fair-code licenses for client installs — same logic as the n8n call.
4. **Scope granularity.** Can the server's tools be scoped (read-only vs
   read-write, per-resource)? If not, it can't be hardened and shouldn't touch
   production data.
5. **Data residency.** Does the server proxy data through a third party, or
   call the provider directly? Direct is on-thesis; any intermediary is a
   quiet re-lock-in.

Rule: prefer official or rubric-passing community servers over vendor-hosted
cloud connectors. The cloud connector is the quiet re-lock-in; the
self-hosted server is the asset.

## 3b — Auth and credentials management (the hard part)

This is the piece clients get most nervous about — *"you want my Gmail
credentials where?"* — and it's the single highest-trust lever in the
engagement. Honest framing: **MCP auth is currently a mess.** The spec moved
to OAuth 2.1 but the enterprise story is widely acknowledged as incomplete
("a mess for enterprise," per Christian Posta's widely-cited writeup). Best
practice is still consolidating around offloading to an external OIDC provider
via OAuth Token Exchange rather than letting the MCP server own the auth flow.

For the reference architecture:

- **Tokens never live in the MCP server's process memory long-term.** Use an
  external secrets store (Doppler, Infisical, or for smaller clients a
  sealed Kubernetes secret / Docker secret) and inject at runtime.
- **Prefer OAuth 2.1 with refresh-token rotation** over static API keys for
  any service that supports it (Gmail, Calendar, Drive, Slack all do).
- **Use the OAuth Device Flow for initial user consent** when the server runs
  headless — this is the supported pattern for server-side MCP without a
  browser callback.
- **Per-client credential isolation.** Each client deployment gets its own
  OAuth app registration and its own secrets namespace. Never share an OAuth
  client across clients — it tangles billing, audit, and revocation.
- **Rotation is a documented deliverable.** Hand the client the rotation runbook
  as part of the engagement, not as an afterthought. The consultancy that
  ships a rotation runbook is the consultancy that gets re-hired.

This is not a solved layer. Flag that honestly in client conversations: you're
implementing the current best practice in a still-standardizing space, and
part of what you're selling is staying ahead of the spec as it matures.

## 3c — Security and scoping model (least privilege)

MCP servers can delete calendar events, send emails, write files. An
unscoped MCP server is a remote-control on the client's accounts handed to
any model that can reach it. The reference architecture enforces
least-privilege as a default, not an option:

- **Read-only by default.** Provision servers with read scopes first; grant
  write scopes only to specific tools that demonstrably need them, and only
  after the client signs off.
- **Progressive scope model.** Follow the spec's tiered pattern — a minimal
  initial scope (e.g. `mcp:tools-basic`) covering only low-risk read/
  discovery operations, escalating to write scopes per tool, per role.
- **Tool-level access control at a gateway if needed.** For multi-tenant or
  higher-sensitivity deployments, put an MCP gateway/proxy in front that
  enforces per-tool, per-agent authorization — this is emerging tooling
  (Aptible, Peta, mcp-sec-audit patterns) rather than a mature category, so
  treat it as a hardening option, not a default.
- **Audit logging on every tool invocation.** Who (which agent / which user),
  what tool, what arguments, what scope was used, what the response was.
  This is the artifact that makes the deployment defensible under SOC 2 /
  GDPR scrutiny and the thing that turns "we installed some MCP servers" into
  "we hardened your tool layer."
- **Pre-install audit.** Run something like `mcpserver-audit` or `mcp-sec-audit`
  against any community server before handing it credentials. Document the
  findings in the engagement handoff. This is a billable step, not a nicety.

This section is the consultancy's moat. Anyone can install an MCP server;
few can install one that's scoped, logged, and audit-ready.

## 3d — Transport model

Two modes, with a clear decision rule:

- **stdio** — local, single-client, spawned per agent session. Simplest, no
  network surface, no shared auth. Use for dev work and single-user setups.
- **Streamable HTTP** — the modern remote standard (the spec moved away from
  raw SSE to Streamable HTTP in 2025). Long-running service, shared across
  multiple clients/sessions, needs process management and its own auth layer.

For the reference architecture's Compose deployment, **Streamable HTTP is the
default** because Inngest/Temporal workflows and LangGraph agents both need
to share the same MCP servers across concurrent runs. stdio is reserved for
local development and for clients with a strict single-session footprint.

## 3e — Custom MCP servers as a service

When a client has a bespoke system — custom CRM, internal API, legacy
database — there's no community server. The official Python SDK
(`modelcontextprotocol/python-sdk`, FastMCP) makes building a custom MCP
server a normal delivery task, not a research project. **These custom servers
are among the highest-value IP you produce:** they're reusable across
engagements with similar systems, they encode domain knowledge as tools any
model can call, and they're the piece clients can't easily build themselves.

Treat custom server builds as a distinct line item in engagements, not
incidental work. A client who commissions a custom MCP server for their
internal API is buying an asset that outlives the engagement — price it that
way.

## 3f — Glue to Layer 2 (orchestration)

MCP tools surface as agent tools through maintained adapters, not hand-rolled
plumbing:

- **LangGraph** — `langchain-mcp-adapters` is the official bridge; it converts
  MCP tools into LangChain/LangGraph-compatible tools and handles multi-server
  fan-out. This is the connective tissue between Layer 2's reasoning tier and
  Layer 3's tool layer.
- **Inngest / Temporal** — call MCP tools as ordinary function calls inside
  workflow steps. No adapter needed; the MCP client SDK is invoked directly.
  This keeps the durable tier decoupled from any specific agent framework.

The clean separation: durable workflows call tools directly via the MCP client
SDK; reasoning agents call tools through the LangGraph adapter. Either way, the
MCP server itself is the same owned asset — the orchestrator just speaks to it
differently.
