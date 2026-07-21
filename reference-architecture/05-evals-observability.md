# Layer 5 — Evals and observability

← [Back to Table of Contents](./README.md)

The previous draft called this "the layer consultancies most underinvest in
and the one that makes the work defensible" — and then spent five lines on it.
That's a contradiction. If evals are the moat, the doc can't hand-wave them.
The expansion corrects two flaws in the draft: it conflated *offline evals*
with *online observability* (different jobs, different tools), and it named
only promptfoo in a category that has legitimate specialization.

The honest reframe: **defensibility comes from a closed loop, not a tool.**
The loop is: golden cases + production traces → CI regression gating →
production failures become new golden cases. Each piece is a billable service;
most small AI companies have none of it. Naming the loop is more valuable
than naming a framework.

## 5a — Offline evals (run before and on every change)

**promptfoo** stays the named default for breadth and red-teaming: CLI, works
with any provider via LiteLLM/OpenAI-compatible endpoints, runs locally,
versionable alongside the flows it tests, strong for adversarial/prompt-
injection testing. But it's not the only tool, and the draft was wrong to
imply it. Reach for specialization when the engagement calls for it:

- **DeepEval** when the client wants pytest-style assertion discipline —
  `@assert_test` decorators, CI/CD integration, treats evals as first-class
  unit tests. Strong fit for teams that already think in test suites.
- **RAGAS** when the work is retrieval-heavy — purpose-built metrics for
  faithfulness, answer relevance, context precision. Don't use it for general
  agent eval; do use it when a RAG pipeline is the engagement.
- **Inspect AI** (UK AISI) when the work is agent trajectories — evaluating
  multi-step tool use, not just final outputs. The serious bet for agent-tier
  evals where the *path* matters as much as the destination.

Don't install all four. Pick by engagement shape. promptfoo covers most small
-client work; escalate to DeepEval/RAGAS/Inspect AI when the client's problem
demands it. The skill is knowing when to escalate, not collecting tools.

## 5b — Online observability (what's happening in production)

Evals catch regressions before ship; observability catches what evals missed
in production. Different job, different layer, and the draft ignored it.

- **Langfuse** as the on-thesis default — open source, self-hostable, does
  tracing + eval + prompt management in one. Lives on the same VPS as
  everything else; data stays with the client. The boring, durable bet.
- **Helicone** as the lighter alternative — also open-source/self-hostable,
  simpler, strong for cost and latency monitoring with zero instrumentation.
  Good for clients who need visibility without a full observability stack.
- **LangSmith / Braintrust** as managed SaaS — capable, but off-thesis for
  residency-sensitive clients because traces (which contain inputs/outputs)
  leave the environment. Same flagging rule as OpenRouter, n8n, and hosted
  vector DBs: allowed if the client knowingly accepts it and the data isn't
  residency-sensitive; never the undocumented default.

The integration point: observability tools consume the LiteLLM gateway's
logs (Layer 1's operational value made concrete) and the MCP audit logs
(Layer 3c). The layers compose — the gateway isn't just routing, it's the
spine of the observability story.

## 5c — The closed loop (the actual moat)

The defensible practice is the loop, not any single tool above:

1. **Golden cases** — curated input/expected-output pairs representing
   critical functionality. Start small (20–50 cases), grow from real
   failures. Each case is versioned alongside the code.
2. **Production traces** — sampled from live runs via the observability layer.
   10–20% of production traffic is the common guidance; 100% for high-stakes
   domains. Traces capture what actually happened, not what you predicted.
3. **CI regression gating** — every prompt or model change runs the golden
   set; a regression blocks ship. promptfoo/DeepEval both integrate with CI;
   the discipline is treating the gate as blocking, not advisory.
4. **Failures become golden cases** — production failures captured by
   observability get added to the golden set with the fixed behavior as the
   expected output. The dataset grows from reality, not speculation.

This loop is the consultancy's moat for three reasons: (a) it's the part
clients most underinvest in, so installing it is high-leverage; (b) it's
sticky — once a client's regression dataset lives in your harness, switching
consultancies means losing that institutional knowledge; (c) it makes every
future change safe to ship, which is the thing clients actually want from AI
engineering and rarely get.

## 5d — LLM-as-judge, used carefully

Many of the evals above use an LLM to score outputs (LLM-as-judge). It's
powerful but has known failure modes: judges prefer verbose outputs, agree
with their own style, drift over time. Use it, but:

- **Calibrate the judge** against human-labeled cases before trusting it.
- **Use a different model for judging than for generation** to avoid
  self-preference.
- **Don't let it be the only signal** — pair with deterministic checks
  (regex, schema validation, exact match) where possible.

This is the kind of nuance that separates "we ran evals" from "we ran evals
you can trust." Worth surfacing in client conversations because it's where
shallow AI consultancies cut corners.

## 5e — Evals as a service offering

The commercial implication worth naming explicitly: this layer is a
distinct, sellable service, not just internal hygiene. Three offerings fall
out naturally:

- **Eval harness installation** — stand up promptfoo/Langfuse, seed the
  golden set, wire CI gating. A fixed-scope engagement.
- **Ongoing eval maintenance** — monthly review of production traces,
  golden-set growth, judge recalibration. A retainer.
- **AI cost-and-reliability audit** — the gateway (Layer 1) + observability
  (5b) together produce the data; this service turns it into recommendations.
  Mentioned earlier as a more saleable entry point than "decouple your model
  access" — evals are the evidence behind the recommendations.

This is the layer where the consultancy's revenue model gets concrete. The
other layers produce infrastructure; this one produces recurring
relationships.
