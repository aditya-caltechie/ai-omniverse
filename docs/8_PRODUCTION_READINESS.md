# Production readiness for LLM & agent systems (generic checklist)

This is a **project-agnostic** version of “enterprise readiness” for AI systems: **LLM apps, RAG, agents, and async pipelines**. It focuses on what you must operationalize before you let real users depend on the system.

---

## 1) Big-picture architecture patterns (two common shapes)

### A) Simple synchronous API

```text
Client -> HTTPS API -> LLM (+ optional tools/RAG) -> Response
```

### B) Production agent pipeline (async)

```text
Client -> HTTPS API -> Job store -> Queue -> Orchestrator/Workers -> Job store -> Client polls/streams results
                      |                 |
                      v                 v
                 Observability      Traces/Scores
```

The async pattern exists for **long-running** agent work, **fan-out** to specialist workers, and **backpressure** when traffic spikes.

---

## 2) What “production-ready” means (definitions)

- **Scalable**: predictable latency and throughput under load; bounded costs; backpressure.
- **Secure**: strong authn/authz; least-privilege; secret hygiene; safe data handling.
- **Observable**: you can answer “what happened?” quickly with logs/metrics/traces.
- **Guarded**: failures degrade safely; validation stops bad writes and unsafe outputs.
- **Operable**: deployable, testable, rollbackable; incidents have runbooks.

---

## 3) Scalability (capacity + cost + safety)

### Must-have decisions

- **Workload split**:
  - **Sync path**: quick request/response (small bounded work).
  - **Async path**: heavy agent workflows (queued).
- **Concurrency strategy**:
  - Reserve capacity for critical services (API + orchestrator).
  - Cap fan-out to avoid thundering herd.
- **Timeouts & budgets**:
  - Request timeout, workflow max duration, max tool calls, max turns.
  - “Token/cost budget per job” for agent loops.
- **Backpressure**:
  - Throttle at the edge and/or enqueue jobs.
  - Reject or defer when queue age/backlog violates SLO.

### “Real production” additions

- **Idempotency**: jobs safe to re-run (dedupe keys, “already completed” checks).
- **Load testing**: measure p95 latency, error rate, throttles, queue backlog, DB pressure.
- **Priority lanes**: separate interactive vs batch queues or priority attributes.

---

## 4) Security (layered defense)

### Baseline controls

- **Authn/authz on every request**
  - Verify tokens; enforce per-tenant isolation on every read/write.
- **Least-privilege IAM**
  - Separate execution roles per service; allow only required actions/resources.
- **Secrets management**
  - No secrets in code or images; rotate credentials; prefer managed stores (Secrets Manager/SSM).
- **Perimeter controls**
  - Rate limiting + strict CORS + input size limits + security headers where applicable.

### Defense-in-depth (when threat model justifies)

- **WAF**: managed rules + bot/rate protections.
- **Private connectivity**: VPC endpoints / private subnets for sensitive calls.
- **Threat detection**: GuardDuty/CloudTrail patterns; anomaly alerts.

### Explicit decisions you should document

- **Data classification**: what counts as PII? retention? deletion?
- **Encryption**: in transit and at rest (and who can decrypt).
- **Rotation cadence**: what rotates and how fast can you revoke a compromised secret.

---

## 5) Monitoring (metrics + dashboards + alarms)

### Minimum signals to chart

- **API health**: request rate, 4xx/5xx, p50/p95 latency, throttling.
- **Compute health**: errors, timeouts, throttles, cold starts (if serverless), saturation.
- **Queue health** (if async): backlog, oldest message age, retries, DLQ > 0.
- **Data store health**: latency, capacity, error rate.
- **Model health**: invocation rate, latency, timeouts, token usage, cost.

### Alarms you want early

- **Error rate** sustained above baseline (API + workers).
- **Timeouts / throttles** > 0 sustained.
- **DLQ messages** > 0 (usually immediate alert).
- **Oldest message age** breaches SLO.
- **Cost guardrails**: daily spend anomaly / token usage spike.

---

## 6) Guardrails (reduce AI failure modes)

Guardrails should exist at **three points**:

```text
Input -> (validate/sanitize) -> Agent -> (budget/loop limits) -> Output -> (validate/score) -> Persist/Show
```

### Must-have guardrails

- **Input validation**: allow-lists where possible; reject malformed identifiers; size caps.
- **Output validation**: schema checks for JSON; required fields; safe fallbacks.
- **Budget limits**: max tool calls, max turns, max response size, max time.
- **Retries**: exponential backoff for transient provider failures (cap attempts).

### Guardrails for higher-risk domains

- **Policy checks**: disclaimers, no unsafe advice, confidence thresholds.
- **Human-readable failure states**: safe error response + diagnostic breadcrumb trail.

---

## 7) Explainability (trust + compliance)

- **Decision rationale**: store “why” alongside structured outputs (not only the final label).
- **Audit trail**: record which components ran, model IDs, prompt versions/hashes, timings.
- **Retention**: define how long you keep traces and whether they include user content.

---

## 8) Observability (tracing + quality signals)

Logs answer **what failed**. Traces answer **where time went** and **what the agent did**.

### Must-have trace attributes

- `request_id` / `job_id`
- `tenant_id` / `user_id` (pseudonymous if needed)
- `component` (api/orchestrator/worker/tool)
- `model_id`, `prompt_version`
- `latency_ms`, `tokens_in/out` (if available)
- `tool_calls` summary

### Quality signals (scores)

- Schema validity pass/fail
- Judge score (sampled or gated)
- Safety policy outcomes (refusal/allowed/escalated)

---

## 9) Production hygiene (often missed)

- **CI/CD**: lint/test, build/package, infra plan/apply, progressive deploys.
- **Environment separation**: dev/stage/prod (isolated data + different rate limits).
- **Backups & recovery**: verify restores at least once.
- **SLOs**: define targets (p95 latency, queue age, error rate) and alert on breaches.
- **Change management**: canary/phased rollout + rollback plan.

---

## 10) “Do this first” checklist (minimal but real)

- **Scale**: throttles + concurrency caps; queue visibility timeouts; DLQ enabled.
- **Security**: auth everywhere; least-privilege; secrets in managed store; size limits.
- **Monitoring**: dashboards + alarms; DLQ alarm; cost budget alert.
- **Guardrails**: input/output validation + budget limits + retries/backoff.
- **Tracing**: end-to-end traceability by job/request ID; sampled quality scoring.

