# AWS deployments for AI/ML backends (practical guide)

This guide explains **how to choose AWS deployment shapes** for projects that are mostly **Python APIs**, **Gradio/Streamlit demos**, or **agent/RAG backends**—without treating “frontend” as a separate product you must master first. The ideas align with the spirit of shared notes like [*AWS Deployment: No Frontend Knowledge Needed*](https://grok.com/share/bGVnYWN5_909a299a-13fd-469d-9e2b-a14c83a0f4b4?rid=a52f275d-638d-4b91-a599-5dbcc2d0b42a) (Grok share—title only retrievable here; treat this doc as the structured checklist).

For how these patterns fit your **course repos** and MLOps sequencing, see **[`6_MLOPS_TRACK.md`](6_MLOPS_TRACK.md)**.

---

## 1. Pick a deployment shape first

| If your app is… | A good default on AWS | Why |
|-----------------|----------------------|-----|
| **Single process**: Gradio, Streamlit, FastAPI + one UI bundle, or a notebook-style API wrapped in `uvicorn` | **Docker → ECR → App Runner** | One **container image**, one **HTTPS URL**, **auto-scaling** and **managed TLS** without you operating Kubernetes. |
| **Split**: static site + HTTP API, or **pay-per-request** API only | **S3 + CloudFront** (optional) + **API Gateway** + **Lambda** | **Scale to zero** on the API side; cheap at low traffic; cold starts matter for LLMs. |
| **Multi-service product**: agents, ingest, vectors, DB, scheduled jobs | **Mix**: **Lambda**, **API Gateway**, **Aurora**, **SQS**, **EventBridge**, **SageMaker**, **S3 Vectors**, **ECR + App Runner**, etc. | Each piece matches **load pattern** and **blast radius** (see [ai-financial-planner](https://github.com/aditya-caltechie/ai-financial-planner)). |

**Rule of thumb:** one container that “does everything” (including Gradio’s built-in UI) → **App Runner**. A platform with separate APIs, workers, and data stores → **compose** serverless + data services.

---

## 2. Pattern A — Docker, ECR, App Runner (Gradio / DS / single backend)

**Best for:** data-science demos, internal tools, **AI/ML backends** where the “UI” is **Gradio** or a **single FastAPI** app serving HTML or JSON.

### What App Runner gives you

- **HTTPS URL** and optional **custom domain** without running nginx yourself.
- **Auto-scaling** based on requests/concurrency (within limits you set).
- Pulls from **ECR**; redeploy when you push a **new image tag**.
- No cluster to manage (contrast with EKS).

### Typical flow

```text
  Dockerfile  →  build image  →  push to ECR  →  App Runner service  →  https://xxxxx.awsapprunner.com
```

### Practical tips

1. **One listening port** — Expose the port Gradio/FastAPI uses (often `7860` for Gradio, `8000` for FastAPI). Set the same in App Runner **port configuration**.
2. **Health check** — Prefer a real **`/health`** route that does not call the LLM (avoid cold-start false negatives and token cost).
3. **Environment variables** — Put API keys in **App Runner environment configuration** or **Secrets Manager** / **SSM Parameter Store** (Parameter Store is fine for many labs; rotate keys in one place).
4. **Image size** — Large CUDA or model layers inflate pull time; for CPU demos, slim base images (`python:3.12-slim`) speed deploys.
5. **Memory / CPU** — LLM or embedding workloads need enough **memory**; under-provisioning causes OOM kills—watch **CloudWatch** logs.
6. **Concurrency** — App Runner scales out; **each instance** may load models—watch **cost** and consider **caching** or a **smaller model** for demos.
7. **Idempotent deploys** — Tag images (`v20260219-1`); roll forward by pointing the service at the new digest/tag.

### ASCII — mental model

```text
                    ┌─────────────────────────────────────┐
                    │  Amazon ECR (private registry)      │
                    │  image: your-app:tag                │
                    └──────────────────┬──────────────────┘
                                       │ pull
                                       v
  Internet ──HTTPS──► ┌────────────────────────────────────┐
                      │  AWS App Runner                    │
                      │  auto-scale · TLS · service URL    │
                      └────────────────┬───────────--──────┘
                                       │
                                       v
                      ┌────────────────────────────────────┐
                      │  Container: Gradio or FastAPI      │
                      │  (UI bundled or API-only)          │
                      └────────────────────────────────────┘
```

**“No separate frontend knowledge”** here means: **Gradio/Streamlit render the UI inside the same service**; your artifact is still **one container** and one deploy target.

---

## 3. Pattern B — Serverless + data plane (example: ai-financial-planner style)

**Best for:** **production-style** apps with **multiple responsibilities**—user-facing API, **async workers**, **vectors**, **relational state**, **scheduled research**.

The **[ai-financial-planner](https://github.com/aditya-caltechie/ai-financial-planner)** (“Alex”) style deployment typically **composes** (see repo `guides/` and `docs/`):

| AWS piece | Typical role |
|-----------|----------------|
| **API Gateway** | Public HTTPS for **ingest** or HTTP APIs; can use **API keys** for service-to-service. |
| **Lambda** | Short **ingest** (embeddings), **schedulers**, adapters; **package size** and **timeout** limits apply. |
| **App Runner** (+ **ECR**) | **Longer-running** “researcher” or agent service that is not a good fit for Lambda duration. |
| **SageMaker** | **Embeddings** or model **inference** with managed scaling. |
| **S3** / **S3 Vectors** | **Artifacts**, **vector** storage, sometimes static assets. |
| **Aurora Serverless** | **Transactional** data (users, jobs, portfolio rows). |
| **SQS** | **Decouple** producers/consumers, absorb spikes, retries. |
| **EventBridge** | **Schedules** (e.g. periodic research). |
| **Bedrock** | Managed models with **IAM**-based access from Lambda/App Runner. |

### Practical tips

1. **Don’t put everything in one Lambda** — Split **sync API** vs **heavy agent** vs **batch**; use **queues** for slow work.
2. **VPC** — Aurora and private endpoints often imply **Lambda in VPC**; that adds **cold start** and **ENI** considerations—budget latency.
3. **Terraform / IaC** — The reference repo uses **separate stacks** per phase; keep **state** isolated and **destroy** sandboxes to save money.
4. **Observability** — Use **CloudWatch** everywhere; add **LangFuse** (or similar) for **agent traces** where the course guides you.

### ASCII — coarse architecture (conceptual)

```text
  Users / schedulers
        │
        ├─► API Gateway ──► Lambda (ingest / API)
        ├─► App Runner ───► Researcher agent (ECR)
        └─► EventBridge ─► Lambda ──► App Runner (scheduled)

        Data & ML:  Aurora ◄──► app   |   S3 / S3 Vectors ◄──► embeddings / RAG
                   SageMaker · Bedrock
```

---

## 4. Pattern C — Static web + API (brief)

**Best for:** Next.js **static export** + **FastAPI** on Lambda ([ai-digital-twin](https://github.com/aditya-caltechie/ai-digital-twin) pattern): **S3** (and optional **CloudFront**) for the UI; **API Gateway** → **Lambda** for `/chat`.  

See **`6_MLOPS_TRACK.md`** §5.2 and upstream **`docs/aws-architecture.md`** for diagrams—this is **not** “Gradio in one container”; it is **two AWS paths** (static vs API).

---

## 5. Decision checklist (before you deploy)

- **Traffic shape:** steady vs bursty vs scheduled?
- **Runtime length:** sub-second API vs minutes of agent work?
- **State:** none, **S3 files**, **vectors**, **Postgres (Aurora)**?
- **Secrets:** API keys only vs **IAM-only** (Bedrock) where possible?
- **Cost:** scale-to-zero (**Lambda**) vs always-on minimum (**App Runner**) vs DB (**Aurora** always costs—pause/destroy labs)?
- **Rollback:** image tags, Lambda aliases, or Terraform state?

---

## 6. CI/CD pointers (practical)

| Step | Container path (ECR + App Runner) | Serverless path (Lambda) |
|------|-----------------------------------|----------------------------|
| Build | `docker build` in CI | Package zip or **container image** for Lambda |
| Push | `docker push` to ECR | Upload to S3 or ECR per Lambda image |
| Deploy | Update App Runner **service** / image tag | Update function **version** or **alias** |
| Test | Hit `/health` then one **smoke** inference | Invoke **test event** + integration test against API Gateway |

Prefer **GitHub Actions** with **OIDC to AWS** instead of long-lived access keys when you move beyond manual deploys.

---

## 7. Related docs in this repo

| Doc | Use it for |
|-----|------------|
| [`6_MLOPS_TRACK.md`](6_MLOPS_TRACK.md) | Week-by-week **project mapping**, Terraform, **reference repos**. |
| [`5_AGENTIC_TRACK.md`](5_AGENTIC_TRACK.md) | **Agent** frameworks (orthogonal to *where* you host). |

---

## 8. References (external)

- [AWS App Runner — How it works](https://docs.aws.amazon.com/apprunner/latest/dg/what-is-apprunner.html) (official)
- [Amazon ECR](https://docs.aws.amazon.com/ecr/) (official)
- [ai-financial-planner](https://github.com/aditya-caltechie/ai-financial-planner) — multi-service AWS example
- [Grok share — “AWS Deployment: No Frontend Knowledge Needed”](https://grok.com/share/bGVnYWN5_909a299a-13fd-469d-9e2b-a14c83a0f4b4?rid=a52f275d-638d-4b91-a599-5dbcc2d0b42a) — inspiration for backend-first framing
