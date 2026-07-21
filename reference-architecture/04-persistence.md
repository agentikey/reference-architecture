# Layer 4 — Persistence

← [Back to Table of Contents](./README.md)

Persistence is where the vendor-neutral thesis gets concrete: data is the
asset clients actually own, so the layer must make residency, portability, and
exit real — not theoretical. The current draft under-specified this. The
expansion maps what data lives where, names the engine convergence that
simplifies deployment, and pressure-tests the vector-storage rule.

## 4a — What data lives where

The architecture produces several distinct data streams, each with different
requirements. Mapping them prevents the common failure of stuffing everything
into one store or, worse, into a hosted service that quietly re-introduces a
dependency:

| Data | Producer | Requirements |
|---|---|---|
| Agent state (checkpoints) | LangGraph | Frequent reads/writes, per-run, swappable backend |
| Workflow run state | Inngest / Temporal | Durable, retryable, the framework owns the schema |
| Application / business data | Your code | Structured, relational, client-owned |
| Vector embeddings | Indexing pipeline | High-dimensional, similarity search, model-determined dimensions |
| MCP audit logs | Layer 3c | Append-heavy, retention policy, compliance-grade |
| Secrets | Layer 3b | External store (Doppler/Infisical), never in the DB |

The insight from this map: most rows can share one engine, which is the
convergence point below.

## 4b — Engine choice and the Postgres convergence

- **SQLite** for local-first / single-tenant client data — file-based, zero-ops,
  portable, ships inside the deployment. The right default for small clients
  and for any deployment where one box is enough.
- **Postgres** as the convergence engine when any of these apply: multi-user,
  concurrent agents, LangGraph in production (needs a real checkpointer, not
  in-memory), Temporal selected in Layer 2 (it requires its own database —
  Postgres is the standard choice; Cassandra is only for huge scale), or
  vector search.

The convergence is the load-bearing insight: **one Postgres instance can serve
as the LangGraph checkpointer, the Inngest/Temporal state store, the
application database, and (with pgvector) the vector store.** That's not
obvious from reading the layers separately, and it's a real operational
simplification — one engine to deploy, back up, upgrade, and hand off. For a
single-VPS client deployment, this is the difference between one service and
four. Name it explicitly when scoping engagements.

When to split: high-throughput audit logging or vector workloads can justify a
dedicated store to keep the primary Postgres healthy. That's a tuning
decision, not a starting architecture.

## 4c — Vector storage strategy

The previous draft's rule ("Postgres + pgvector, avoid hosted vector DBs") was
decent but under-justified and missed the middle option. Three tiers, honest
about trade-offs:

- **pgvector (default).** Lives inside the Postgres instance from 4b, zero new
  moving parts, handles retrieval workloads up to ~low millions of vectors.
  The on-thesis default for most small-AI-company engagements because it adds
  no new dependency.
- **Self-hosted dedicated vector DB (Qdrant, Milvus, Weaviate).** The middle
  option the previous draft missed. Justified when vector workloads outgrow
  pgvector — high dimensionality, tens of millions+ of vectors, specialized
  filtering. Still self-hosted, still on-thesis; just a dedicated engine.
- **Hosted vector DB (Pinecone, Weaviate Cloud).** Off-thesis by default —
  introduces a vendor in the data path for the client's most sensitive asset.
  Accept it only when a client specifically wants managed infra *and* the data
  isn't residency-sensitive, and flag it as a known dependency the same way
  you'd flag OpenRouter or n8n. The principle holds: off-thesis dependencies
  are allowed if documented, not if hidden.

## 4d — Embeddings as a persistence-adjacent decision

pgvector is storage; embeddings need a model, and that choice is a persistence
decision because it determines vector dimensions and storage format. This
crosses Layer 1 (model access) and Layer 4 (persistence) — worth naming so the
seam doesn't hide a gotcha:

- **Hosted embedding APIs** (OpenAI text-embedding-3, Voyage) — easy, high
  quality, but the text leaves the client's environment. Off-thesis for
  residency-sensitive data.
- **Local embedding models** (bge-m3, nomic-embed-text via Ollama) — zero cost,
  text never leaves the box, on-thesis for residency. Quality is competitive
  enough for most retrieval tasks; accept the latency trade-off.

The choice is sticky: changing embedding models means re-indexing everything,
so pick deliberately and document the dimensions in the schema. This is the
kind of decision that's cheap to get right early and expensive to migrate
later — exactly the work a consultancy should front-load for a client.

## 4e — Data residency and the local-first story

This is the selling point, not just a technical detail. The persistence
choices above make residency concrete and defensible:

- **SQLite** = data literally never leaves the client's machine or VPS. The
  strongest possible residency story — the file is the database.
- **Postgres on the same VPS** = no third party in the data path. Strong
  residency story, scales further than SQLite, still self-hosted.
- **Local embeddings via Ollama** = even the indexing pipeline stays on-box.

The pitch to a residency-sensitive client: *your data, your embeddings, and
your agent state all live on one box you control. Nothing in this stack
requires your text to touch a third party.* That's a concrete claim most AI
consultancies can't make because their stack assumes hosted everything. It's
worth leading with for clients in regulated industries or with data-residency
clauses in their own contracts.

## 4f — Portability, backup, and migration

The thesis is "assets you own" — so the data must be portable, backed up, and
evolvable. This is the vendor-neutral promise made operational:

- **Portability is a feature, not an afterthought.** SQLite is a file (trivially
  copyable). Postgres uses `pg_dump` — standard, scriptable, no proprietary
  export format. A client who can't take their data and leave doesn't own it;
  this layer ensures they can.
- **Backup is a documented deliverable.** Ship a backup script (or Compose-side
  `pg_dump` cron) and a restore runbook as part of the engagement. Same logic
  as the credential rotation runbook in 3b — the consultancy that ships the
  runbook is the consultancy that gets re-hired.
- **Schema management via Alembic.** Hand the client a migration tool, not a
  pile of manual SQL. Versioned, reviewable, runnable on upgrade — the same
  defensible-IP argument as Python orchestration over visual flows.

This section is the operational delivery of the whole architecture's promise. If a
client asks "what happens if we stop working with you?" — the answer lives
here: you get your data, in standard formats, with the scripts to restore it,
and the migrations to evolve it. No vendor relationship required to keep
running.
