# Design principles — Agentikey's operating thesis

← [Back to Table of Contents](./README.md)

Agentikey's thesis is one sentence: **own the layers that matter; rent the ones
that don't, and know the difference.** A small AI company's stack is usually a
patchwork of rented surfaces — a hosted model, a managed agent platform, a
visual automation tool, a hosted vector database, a SaaS observability
dashboard. Some are fine; most are inherited without anyone asking which. The
principles below are the facets of owning the layers that matter to the
business and renting the rest deliberately, not by default.

1. **Own the workflow, not the vendor.** Models are pluggable; orchestration
   and tooling are owned, not rented. A vendor can be swapped without rewriting
   the IP. Nothing a client runs depends on a single AI vendor — including us.
   A stack you can't reroute is a stack you don't own.

2. **Own the data path that matters.** Self-hosted tooling where the data is
   sensitive, auditable, or residency-bound; cloud connectors where the data
   isn't. The same MCP server runs for Agentikey and for a client — only the
   credentials and target data change. The distinction is knowing which path
   matters: a client list in Airtable is a deliberate rental; a zoning
   ordinance corpus leaving the box is a line you don't cross. We install
   ownership where it matters and document rentals where it doesn't, so
   nothing is inherited by accident.

3. **Own the architecture's shape.** Separate concerns; don't force one tool
   to do two jobs. Business operations (scheduled, retryable, stateful) and
   agent reasoning (multi-step, branching) have different shapes — one
   framework doing both is one framework doing both badly. One storage engine
   where it converges, split where it doesn't. Clean boundaries are a
   deliverable, not a preference.

4. **Own the skills, not the framework.** Bet on durable skills — planning,
   state, tools, evals — and treat any specific framework as replaceable. The
   client leaves with assets and skills that outlive the engagement, not a
   dependency on a tool we happen to favor today. Frameworks churn; the
   pattern survives.

5. **We own it too.** Every pattern here runs inside Agentikey before it
   reaches a client. The reference architecture is live, versioned, refined
   by real engagements — not a brochure. When a client asks "does this
   actually work?" the answer is "we run it ourselves," and that's verifiable,
   not a slogan.
