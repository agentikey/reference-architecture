# Layer 1 — Model access (pluggable)

← [Back to Table of Contents](./README.md)

The model layer isn't one decision; it's three shapes, and the distinction
that matters for a vendor-neutral thesis is inside the third. Choosing well
here is the difference between *owning* the model layer and *renting* it.

**A. No gateway — direct SDKs / native OpenAI-compatible endpoints.**
Anthropic and Google now both ship OpenAI-compatible endpoints, so all three
major providers can be hit through a single SDK with provider-specific base
URLs. Most control, no new vendor, most integration code. The right default
for clients who want zero intermediary and are willing to maintain the small
amount of per-provider wiring themselves. Two years ago the argument for a
unified gateway was load-bearing; with native OpenAI-compatible endpoints,
it's less so.

**B. Self-hosted gateway — plumbing, no new model vendor. (on-thesis default)**
Self-hosted LiteLLM, Cloudflare AI Gateway, Portkey self-hosted, Bifrost, or
Kong. You keep direct billing relationships with providers; the gateway does
routing, failover, caching, cost tracking, rate limiting, and observability.
This is *infrastructure*, not a counterparty — nothing in the path takes a
margin or controls your model access. **Self-hosted LiteLLM is the named
default** for the reference architecture because it's open source, broadly
adopted, and calls providers directly. The alternatives in this tier are
legitimate substitutes; pick by operational fit, not theology.

**C. Reseller / aggregator — a new vendor in the path. (off-thesis for clients)**
OpenRouter is the clear example (300+ models, one API key, one bill);
Portkey managed shades into this. Trivial model swap and fast on-ramp, but
you've introduced a counterparty that takes margin and controls pricing,
terms, and access. **This is the kind of dependency you help clients
*remove*, not install.** Fine as a personal velocity on-ramp; if it shows up
in a client architecture, flag it as a known dependency to retire, not a
permanent fixture.

---

**Decision rule:** for Agentikey's own stack, default to **B (self-hosted
LiteLLM)**, with **A** as the minimal-overhead option where the operational
features of a gateway aren't worth the moving parts. Use **C (OpenRouter)**
only to bootstrap a flow fast, then migrate to B once the pattern is proven.
For clients, recommend B or A and treat any C dependency as a flagged item
in the audit.

**Why the gateway value has reframed around operations, not interface
unification.** The old pitch for a gateway was "one interface so you're not
maintaining three SDKs." Native OpenAI-compatible endpoints weakened that
argument. The remaining value is operational and that's actually a stronger
consultancy pitch: failover between providers, per-task cost tracking,
rate-limit governance, response caching, and observability across the whole
model surface. Small AI companies bleed money on model costs they can't see
and lose reliability they can't diagnose — the gateway-as-control-plane
framing lands harder than "call different models," and it's the part clients
genuinely can't justify building for one stack but will pay to have
installed and hardened.

---

The model is a component, chosen per task by cost/quality/latency/residency.
Claude stays a default for hard reasoning; cheaper models handle routing,
classification, summarization. Local inference (Ollama) is reserved for
client data that can't leave device, or cost-zero inference for low-stakes
steps — a residency tool, not a daily driver.
