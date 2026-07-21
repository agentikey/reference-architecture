# Layer 6 — Deployment

← [Back to Table of Contents](./README.md)

This layer makes the whole architecture real. The Compose file isn't a config
detail — it's the handoff artifact, the thing a client actually receives and
runs. Treating it as a versioned, tested product (not a one-off) is what
turns the reference architecture into a repeatable engagement. The expansion
makes the inventory complete, the costs concrete, and the limits honest.

## 6a — The complete service inventory (one box)

The draft's old list ("Inngest, LiteLLM, MCP servers, SQLite/Postgres,
promptfoo") is now incomplete — the expanded layers above added components.
The full single-box picture, by layer:

| Service | Layer | Purpose |
|---|---|---|
| LiteLLM proxy | 1 | Model gateway, routing, cost tracking |
| Redis | 1 | LiteLLM caching + rate-limiting (production checklist item) |
| Inngest (or Temporal) | 2 | Durable workflow execution |
| LangGraph runtime | 2 | Agent execution (runs inside the Inngest/Temporal service or its own) |
| MCP servers (Streamable HTTP) | 3 | Tool access — Google Workspace, GitHub, custom |
| Postgres + pgvector | 4 | Convergence store: app data, agent state, workflow state, vectors |
| Langfuse | 5b | Tracing + observability + prompt management |
| promptfoo (CI runner) | 5a | Offline evals, runs on demand or in CI |
| Secrets store (Doppler/Infisical) | 3b | Credential management, injects at runtime |
| Caddy or Traefik | 6c | Reverse proxy, TLS, routing |

The Postgres convergence (4b) is doing real work here: without it, this table
would have four database engines instead of one. That's the operational
payoff of the layer-4 design — one engine to deploy, back up, and upgrade.

## 6b — Host choice and cost reality

Clients ask "what will this cost to run?" — the answer is a selling point,
not a footnote:

- **Hetzner** — value king, €5-20/mo gets a box that comfortably runs this
  stack (4-8 vCPU, 8-16GB RAM). Best for cost-sensitive clients; datacenters
  in EU/US.
- **Fly.io** — ~$5-15/mo, easier deploy story (native Compose-ish via
  `fly.toml`, good CLI). Best when you want less ops surface and can accept
  slightly higher cost.
- **DigitalOcean / Linode** — $5-24/mo, familiar to most clients, good for
  those who already have an account or want US-based support.

The concrete pitch: **a vendor-neutral AI stack that runs the whole
architecture for $10-30/month** — less than a Claude Pro subscription, with no
per-seat or per-call vendor margins on top. For small AI companies bleeding on
SaaS bills, this number lands. Document it in proposals.

For Agentikey's own stack: one Hetzner VPS. For clients: their existing infra
if they have it, or a VPS provisioned as part of the engagement. The Compose
file runs identically either way — that's the portability promise made real.

## 6c — Networking, TLS, and secrets in Compose

The boring-but-real ops layer clients need to actually run in production:

- **Reverse proxy with automatic TLS.** Caddy (simpler config, auto-Let's-
  Encrypt) or Traefik (more features, steeper curve). One of them fronts every
  HTTP service and terminates TLS. Don't expose service ports directly.
- **Internal network isolation.** Compose networks segregate the public-facing
  proxy from internal services (Postgres, Redis, MCP servers) that should
  never be reachable from the internet. The default posture: only Caddy/
  Traefik and the LiteLLM API endpoint are public; everything else is internal.
- **Secrets injection.** Connects to 3b's credential management. Three options
  by maturity: (1) Doppler/Infisical sidecar or `doppler run` as the process
  entrypoint — the on-thesis default, secrets never touch disk; (2) Docker
  secrets for the small-client case — file-based, mounted, simpler but less
  rotation-friendly; (3) plaintext `.env` files — only for local dev, never
  production. Document which a client is getting and why.
- **MCP server auth.** Streamable HTTP MCP servers (3d) need their own auth
  layer when they're networked — an API key or token at the proxy level, or
  mTLS for higher-security deployments. Don't leave a networked MCP server
  unauthenticated even on an internal network.

## 6d — The Compose file as handoff artifact (the commercial load-bearer)

This is the most commercially important sub-section in the whole doc. The
Compose file + README + eval harness isn't a deployment detail — it's the
deliverable, and treating it as a versioned product is what makes the
consultancy repeatable:

- **Version it.** The Compose template, the LiteLLM config, the MCP server
  selections, and the eval harness live in a repo (yours, mirrored to the
  client's). Every client engagement is a branch or a tagged fork, not a
  copy-paste. When you improve the template, future clients inherit the
  improvement.
- **Test it.** A `docker compose up` on a fresh VPS should produce a working
  stack with no manual steps beyond secrets injection. Test this regularly —
  an untested handoff artifact rots fast. This is the CI for the deployment
  layer, paralleling the eval harness for the application layer.
- **Document it.** The README isn't optional. It covers: what each service
  does, how to rotate secrets (3b), how to back up and restore (4f), how to
  run the evals (5a), how to read the observability dashboard (5b), and the
  escalation signals for when to move off Compose (6e). A client who can't
  operate the stack without calling you doesn't own it.
- **License it deliberately.** Decide whether the template is yours (client
  gets a deployable artifact but not the reusable template) or theirs (full
  IP transfer). This is a commercial decision, not a technical one — name it
  in the engagement contract, not the README.

The principle: **the handoff artifact is the product.** Everything else in
this architecture exists to make this artifact real, owned, and operable by
the client.

## 6e — The escalation path to Kubernetes (when Compose isn't enough)

Compose is the default, not the ceiling. Name the threshold honestly so
clients know when they've outgrown it, and so the consultancy has a natural
upgrade engagement rather than a hidden gotcha:

- **Multi-node scaling** — when one box can't handle the load (high concurrent
  agent runs, large inference volume) and you need horizontal scaling.
- **Zero-downtime rolling updates** — when taking the stack down for updates
  is unacceptable (production SLAs, always-on agents).
- **Enterprise security controls** — RBAC, network policies, audit at the
  orchestration layer (beyond what MCP audit logging in 3c covers).
- **Multi-region or HA** — when the client's requirements include geographic
  redundancy or failover.

Migration takes 2-4 weeks for a container-familiar team (Compose services
become Kubernetes deployments; the Compose file becomes Helm charts or
Kustomize manifests). The good news: because the architecture is already
containerized and model-agnostic, the migration is operational, not a
rewrite. LiteLLM ships official Helm charts; Temporal has Helm charts; the
MCP servers and LangGraph runtimes move as-is.

The framing for clients: **Compose is the right starting architecture for
~90% of small-AI-company engagements. Kubernetes is an upgrade engagement,
not a starting point.** Don't let clients over-engineer early; do have the
upgrade path ready when they grow. This is also where the consultancy's
retainer model compounds — the client who grows past Compose is the client
who keeps paying you to evolve the stack.

## 6f — Maintenance cadence and operational ownership

The "honest trade-offs" section flags that you carry maintenance. Layer 6
makes that concrete:

- **Image updates.** LiteLLM, Langfuse, Inngest, Postgres, Redis, and MCP
  servers all release updates. Cadence: monthly review, quarterly patch
  unless a security CVE forces faster. Document this as a billable retainer,
  not incidental work.
- **MCP server rotation review.** When 3b's rotation cadence fires, someone
  has to run it and verify the new credentials work end-to-end. This is the
  most operationally tedious part of the stack and the part clients most
  often let slip — exactly the work a retainer covers.
- **Eval golden-set growth.** 5c's closed loop requires ongoing attention —
  reviewing production traces, adding failure cases, recalibrating judges.
  Monthly cadence; billable.
- **Backup verification.** 4f's runbook is worthless if no one tests restores.
  Quarterly restore drill; document the result.

The commercial implication, paralleling 5e: **operational ownership is a
retainer, not a handoff.** The architecture is designed to be operable by the
client — but the ongoing maintenance work (updates, rotation, eval growth,
backup drills) is where the consultancy relationship continues past the
initial engagement. Price it as such.
