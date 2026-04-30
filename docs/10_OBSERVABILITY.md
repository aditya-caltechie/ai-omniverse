# Observability for LLM/agent systems (traces + spans + scores)

This doc explains **observability** for LLM apps and multi-agent pipelines: tracing, tool calls, model usage, and quality signals. It complements **monitoring** (`9_MONITORING.md`) and **logging** (structured logs).

---

## 1) What observability means for agentic AI

For agentic systems, you typically need to answer:

- Which run produced this output?
- Which tool calls happened (and with what arguments)?
- Where did time go (planning vs tool vs model)?
- Which model and prompt version were used?
- What quality/safety score did we assign?

**Key idea:** logs are linear; agent execution is a **tree**. Tracing captures the tree.

---

## 2) Trace model (generic)

```text
Trace (one job/run)
  ├─ Span: planner/orchestrator
  │    ├─ Span: tool call A
  │    ├─ Span: model generation
  │    └─ Span: tool call B
  ├─ Span: specialist worker #1
  ├─ Span: specialist worker #2
  └─ Span: judge/eval (score + rationale)
```

### Common entities

- **Trace**: one end-to-end request or job.
- **Span**: a step inside the trace (model call, tool call, DB write, judge).
- **Event**: milestone (“job queued”, “fallback used”).
- **Score**: numeric quality/safety/correctness signal attached to a trace/span.

---

## 3) Why traces beat logs alone (especially for multi-agent)

Logs help with:

- crashes/exceptions
- AWS permission failures
- “job_id processed”

But logs are hard when:

- execution is split across many services
- you need nested breakdowns of internal steps
- you need token/cost attribution and per-step latency

**Best practice:** use both.

- **Cloud monitoring/logs**: infra reliability, throttles, timeouts, IAM issues.
- **Tracing UI**: agent behavior, tool/model steps, quality scoring, regressions.

---

## 4) Where to instrument (what to wrap)

Instrument the boundaries where meaning changes:

- **API handler**: request start/end, auth result, input size, response status.
- **Workflow/orchestrator**: plan selection, fan-out, retries, queue enqueue.
- **Workers/specialists**: model call spans, tool call spans, DB write spans.
- **Judges/evals**: judge model call + score span/event.

### If you’re on serverless

Serverless runtimes can terminate quickly; ensure your tracer **flushes** exports before exit.

---

## 5) Naming and IDs (how you make traces searchable)

### Stable identifiers (minimum)

- `request_id` (API edge request)
- `job_id` (async pipeline key)
- `tenant_id` (or org/account)
- `user_id` (pseudonymous if needed)

### Metadata to include

- `model_id`, `provider`, `region`
- `prompt_version` (or prompt hash)
- `tool_names` used
- `retry_count`
- `tokens_in`, `tokens_out` (if available)

---

## 6) Quality signals: scores, events, and gates

Scores make subjective quality observable.

### Common score types

- **Schema score**: output validates (1/0) or % fields correct.
- **Judge score**: 0–1 or 0–100 with rubric.
- **Safety score**: policy compliance; injection resistance.
- **Groundedness score** (RAG): answer supported by retrieved context.

### Event examples (useful for ops)

- “fallback used”
- “budget exceeded”
- “tool error retried”
- “job timed out”

---

## 7) Practical troubleshooting workflow (trace-first)

When a user reports “bad output”:

1. Find by **job_id** / time window.
2. Inspect trace tree: locate slow spans, tool errors, retries.
3. Check judge score and rationale (if present).
4. Compare against a “good” run to spot divergence (different tools? different prompts? more retries?).
5. Turn the failure into an **eval case** (`12_EVALS.md`).

---

## 8) Privacy and retention (don’t ignore)

- Decide what you store: raw prompts/responses vs hashes/summaries.
- Redact or tokenize sensitive data (PII) before exporting.
- Set retention windows and access controls.

---

## 9) Observability maturity ladder

- **Level 0**: logs only.
- **Level 1**: traces around API + model calls.
- **Level 2**: tool spans + job IDs across services.
- **Level 3**: judge scores + regression dashboards.
- **Level 4**: trace-driven dataset mining and automated release gates.

