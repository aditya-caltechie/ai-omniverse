# AWS deployments for AI/ML backends (practical guide)

This guide explains **how to choose AWS deployment shapes** for projects that are mostly **Python APIs**, **Gradio/Streamlit demos**, or **agent/RAG backends**—without treating “frontend” as a separate product you must master first. The ideas align with the spirit of shared notes like [*AWS Deployment: No Frontend Knowledge Needed*](https://grok.com/share/bGVnYWN5_909a299a-13fd-469d-9e2b-a14c83a0f4b4?rid=a52f275d-638d-4b91-a599-5dbcc2d0b42a) (Grok share—title only retrievable here; treat this doc as the structured checklist).

---

## 1. Pick a deployment shape first

| If your app is… | A good default on AWS | Why |
|-----------------|----------------------|-----|
| **Single process**: Gradio, Streamlit, FastAPI + one UI bundle, or a notebook-style API wrapped in `uvicorn` | **Docker → ECR → App Runner** | One **container image**, one **HTTPS URL**, **auto-scaling** and **managed TLS** without you operating Kubernetes. |
| **Split**: static site + HTTP API, or **pay-per-request** API only | **S3 + CloudFront** (optional) + **API Gateway** + **Lambda** | **Scale to zero** on the API side; cheap at low traffic; cold starts matter for LLMs. |
| **Multi-service product**: agents, ingest, vectors, DB, scheduled jobs | **Mix**: **Lambda**, **API Gateway**, **Aurora**, **SQS**, **EventBridge**, **SageMaker**, **S3 Vectors**, **ECR + App Runner**, etc. | Each piece matches **load pattern** and **blast radius** (see [ai-financial-planner](https://github.com/aditya-caltechie/ai-financial-planner)). |

**Rule of thumb:** one container that “does everything” (including Gradio’s built-in UI) → **App Runner**. A platform with separate APIs, workers, and data stores → **compose** serverless + data services.

---

## 2. Gradio & Python LLM/agent apps on AWS (no React required)

This section is the **clearest path** for **[LLM track](2_LLM_CORE_TRACK.md)** and **[agentic track](5_AGENTIC_TRACK.md)** work: you keep **the same Gradio or FastAPI app** you use in notebooks and Hugging Face Spaces. **No React, no Node.js, no static build step** unless you *choose* a split frontend later.

Below are **production-friendly patterns** that work with current AWS docs and tutorials (as of **2026**). Order is roughly **easiest → more DIY**.

### 2.1 Compare options at a glance

| Approach | What you manage | Traffic pattern | Good fit |
|----------|-----------------|-----------------|----------|
| **App Runner** | Dockerfile + (optional) Git or **ECR** | Steady or moderate; **auto-scale**, TLS included | **Easiest managed** path for Gradio/FastAPI—**one HTTPS URL**. |
| **Lambda + container image** + [**Lambda Web Adapter**](https://github.com/awslabs/aws-lambda-web-adapter) + [**Function URL**](https://docs.aws.amazon.com/lambda/latest/dg/urls-invocation.html) | Container image; adapter translates HTTP → Lambda | **Bursty**, scale-to-zero | Cheap when idle; watch **cold starts** for heavy LLM loads. |
| **EC2** | OS, security group, process or Docker | Learning, fixed server | **Fastest to understand**; add **ALB + TLS** (or **CloudFront**) when you need polish. |
| **ECS / Fargate** | Task defs, cluster, networking | Container teams, more control than App Runner | “Serverless containers” without managing EC2 for each box. |
| **Elastic Beanstalk** | Upload app; AWS stacks layers | Classic Python web apps | Less fashionable but still valid **managed platform**. |
| **Lightsail** | Simple VPS + fixed pricing | Tiny side projects | Lowest **cognitive** overhead, fewer integrations than full AWS. |

**Same Gradio code** you use on [Hugging Face Spaces](https://huggingface.co/docs/hub/spaces) can run on these paths: bind to `0.0.0.0`, correct **port**, **API keys** from env.

### 2.2 AWS App Runner (usually the easiest managed option)

1. **Containerize** with a small **Dockerfile** (Gradio default port often **7860**).
2. Create an App Runner **service** from **ECR** (or connect a **source repo** if you use AWS’s build path).
3. App Runner **handles** scaling, load balancing, **TLS/HTTPS**, and rolling deploys.

**Cost (rule of thumb):** a **small always-on** service is often on the order of **~$0.12/day** in many regions/configs—verify with the **AWS pricing calculator** and your CPU/memory settings.

**Best when:** you want a **stable URL** and predictable ops without running Kubernetes.

For what App Runner **already includes** (vs optional add-ons like API Gateway or S3), see **§3 Pattern A — “Batteries included.”**

### 2.3 Lambda + container image + Function URL (serverless & cheap for bursty use)

1. Package your HTTP app (including Gradio) as a **Lambda container image**.
2. Add **[AWS Lambda Web Adapter](https://github.com/awslabs/aws-lambda-web-adapter)** so Lambda can run an ordinary web process.
3. Enable a **[Lambda Function URL](https://docs.aws.amazon.com/lambda/latest/dg/urls-invocation.html)** — **HTTPS endpoint** without API Gateway.

**Best when:** usage is **sporadic** (agents called occasionally); you pay roughly **per request** and idle time is cheap. **Watch:** **cold start** time if the container loads large models.

### 2.4 EC2 (quick and transparent)

1. Launch an instance; install Python + deps (or run **Docker** on the instance).
2. One Gradio change: `demo.launch(server_name="0.0.0.0", server_port=7860)` (or your port).
3. Open the **security group** to your IP (or ALB). Optionally put **Nginx**, **Application Load Balancer**, or **CloudFront** in front for TLS and stability.

**Best when:** you’re learning AWS networking end-to-end or need **full control** on a single VM.

### 2.5 Other AWS options (short)

- **ECS on Fargate** — Run the same Docker image as App Runner with **more knobs** (VPC, service mesh–style setups).
- **Elastic Beanstalk** — Upload a Python web app; AWS provisions capacity. Good if you want a **PaaS feel** on AWS.
- **Lightsail** — Fixed bundles for **tiny** projects; fewer services to learn than the full console.

### 2.6 When would you actually need React or Node on AWS?

You **do not** need them for **LLM track** or **agentic track** demos in Python. Consider a **JavaScript frontend** when:

| Situation | Why JS/React might enter |
|-----------|---------------------------|
| **Highly custom UI** | Beyond what **Gradio** / **Streamlit** components offer out of the box. |
| **Deliberate split** | Product/team wants **separate** frontend and API releases, design systems, or mobile clients. |
| **Amplify full-stack** | You already chose **AWS Amplify** and a React/Vue app for auth + hosting integration. |

Even then it is a **product choice**, not a prerequisite. Many production AI tools ship **Gradio or Streamlit** (or a thin FastAPI + templates) and serve customers fine.

### 2.7 Is Gradio “enough” for production on AWS?

Often **yes**, for internal tools, MVPs, and specialist UIs.

- **Built-ins:** auth patterns (including **OAuth** in newer flows), **queues** for long jobs, **file uploads**, and rich components.
- **Scale:** at higher concurrency you may configure **stickiness** (e.g. **client IP** on a load balancer) when multiple replicas need session affinity—depends on how you store session state.
- **Evolution:** you can later expose **Gradio as an API** or put a custom SPA **in front of** the same backend—**without** rewriting the agent/LLM logic first.

### 2.8 Hugging Face Spaces vs AWS

| | Hugging Face Spaces | AWS (App Runner, Lambda, EC2, …) |
|--|--------------------|-----------------------------------|
| **Ops** | Near-zero infra | You choose **networking, IAM, scaling** |
| **Control** | Platform limits | **VPC**, **Bedrock**, **SageMaker**, private data paths |
| **Cost model** | Freemium / subscription tiers | **Pay for what you provision**—optimize with scale-to-zero vs always-on |
| **Same code** | Yes — export/port your app | Yes — **Docker + env vars** |

### 2.9 Bottom line (LLM & agent workflows)

- You **do not** need “fundamentals of React/Node” to **ship** Python LLM or agent apps on AWS.
- What **does** help: **Docker**, **ports**, **environment variables**, **HTTPS**, and basic **CloudWatch** log checking.
- **Fastest paths today** for a Gradio AI agent: **App Runner** or **Lambda + container + Web Adapter + Function URL**—both have **short tutorials** once Docker works locally (often **under 30 minutes** for the cloud wiring).

---

## 3. Pattern A — Docker, ECR, App Runner (Gradio / DS / single backend)

**Best for:** data-science demos, internal tools, **AI/ML backends** where the “UI” is **Gradio** or a **single FastAPI** app serving HTML or JSON.

### What App Runner gives you

- **HTTPS URL** and optional **custom domain** without running nginx yourself.
- **Auto-scaling** based on requests/concurrency (within limits you set).
- Pulls from **ECR**; redeploy when you push a **new image tag**.
- No cluster to manage (contrast with EKS).

### Batteries included: you do not need the rest of the AWS catalog by default

You **do not** need **API Gateway**, **S3**, **Route 53**, or **DynamoDB** (or any database) **just because** you are running on **App Runner**. App Runner is deliberately **batteries-included** for this kind of **Python / Gradio** (or single-process FastAPI) app. The service already gives you:

| Built-in | What it means for you |
|----------|------------------------|
| **Container runtime** + **auto-scaling** | Your image runs as a **service** that scales with traffic—no separate compute orchestration. |
| **Load balancing** | Traffic is spread across instances App Runner creates for you. |
| **HTTPS** | **Managed TLS** / free certificate for the default **App Runner** URL (and custom domains when configured). |
| **Public URL** | A stable **HTTPS endpoint** out of the box—no API Gateway in front required for a standard web app. |
| **Observability** | **CloudWatch** logging and metrics integrated with the service. |

**Everything else is optional** and depends on what **your** app needs in production (e.g. add **S3** only if you offload uploads or static assets; add **Route 53** only for a **custom domain** you own; add a **database** only if your app persists data outside the container).

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

## 4. Pattern B — Serverless + data plane (example: ai-financial-planner style)

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

## 5. Pattern C — Static web + API (brief)

**Best for:** Next.js **static export** + **FastAPI** on Lambda ([ai-digital-twin](https://github.com/aditya-caltechie/ai-digital-twin) pattern): **S3** (and optional **CloudFront**) for the UI; **API Gateway** → **Lambda** for `/chat`.  

See **`6_MLOPS_TRACK.md`** §5.2 and upstream **`docs/aws-architecture.md`** for diagrams—this is **not** “Gradio in one container”; it is **two AWS paths** (static vs API).

### Why [ai-digital-twin](https://github.com/aditya-caltechie/ai-digital-twin) carries a larger AWS stack than a Gradio-only app

The twin project’s own README and **`docs/aws-architecture.md`** describe a **split**: a **built frontend** (static files) and a **serverless API** (Lambda). A minimal Gradio + App Runner app (here contrasted with **ai-deals2buy** as a previous, simpler shape) only needs **one long-lived container**. The table below matches that intent—**not** a feature-by-feature audit of either repo.

| Aspect | **ai-deals2buy** | **ai-digital-twin** and **ai-financial-planner** | **Why the extra services?** |
|--------|--------------------------------------|--------------------------------|-----------------------------|
| **UI** | Pure **Gradio** (Python renders the UI live) | **Next.js + React + TypeScript** (custom chat UI) | Custom UI is shipped as **static assets** → host on **S3** (+ optional **CloudFront** for HTTPS/CDN). |
| **Backend** | Monolithic Gradio + background timer in one process | **FastAPI** only (API server; no bundled Gradio UI) | Fits **serverless**: **Lambda** + **Mangum** (ASGI) instead of an always-on process. |
| **Memory / storage** | Local files + **Chroma** (e.g. mounted via **EFS** on the container) | Local **`memory/`** in dev → **S3** in prod (`USE_S3=true`) | Prod conversations are **object storage**—no shared filesystem required for Lambda. |
| **Deployment model** | Long-running **container** (**App Runner**) | **Serverless**: **Lambda** + **API Gateway** | Lambda is not directly on the public internet; **API Gateway** exposes **HTTPS** routes (`/chat`, `/health`, …). |
| **Custom domains** | Optional (default **App Runner** HTTPS URL is often enough) | **Clean UX**: separate origins for **static site** vs **API** | **Route 53** + **ACM** for HTTPS on **CloudFront** and the **API** endpoint. |
| **Infra as code** | Simpler surface (e.g. **`apprunner.yaml`** / console paths) | **Terraform** for **many** resources (often **8+** in Day-2 guides) | **Repeatable** Day-2 deploy: same stack for you/teammates; destroy/recreate labs safely. |

**Takeaway:** the twin stack is larger because it is **two publish paths** (static site + API), **stateless API compute** (Lambda), and **S3-backed** session storage—not because the LLM logic is inherently more complex than Gradio.

---

## 6. Decision checklist (before you deploy)

- **Traffic shape:** steady vs bursty vs scheduled?
- **Runtime length:** sub-second API vs minutes of agent work?
- **State:** none, **S3 files**, **vectors**, **Postgres (Aurora)**?
- **Secrets:** API keys only vs **IAM-only** (Bedrock) where possible?
- **Cost:** scale-to-zero (**Lambda**) vs always-on minimum (**App Runner**) vs DB (**Aurora** always costs—pause/destroy labs)?
- **Rollback:** image tags, Lambda aliases, or Terraform state?

---

## 7. CI/CD pointers (practical)

| Step | Container path (ECR + App Runner) | Serverless path (Lambda) |
|------|-----------------------------------|----------------------------|
| Build | `docker build` in CI | Package zip or **container image** for Lambda |
| Push | `docker push` to ECR | Upload to S3 or ECR per Lambda image |
| Deploy | Update App Runner **service** / image tag | Update function **version** or **alias** |
| Test | Hit `/health` then one **smoke** inference | Invoke **test event** + integration test against API Gateway |

Prefer **GitHub Actions** with **OIDC to AWS** instead of long-lived access keys when you move beyond manual deploys.

---

## 8. Related docs in this repo

| Doc | Use it for |
|-----|------------|
| [`2_LLM_CORE_TRACK.md`](2_LLM_CORE_TRACK.md) | **LLM engineering** curriculum—RAG, eval, fine-tuning arc; **Gradio**-style UIs map to **§2** of this doc. |
| [`5_AGENTIC_TRACK.md`](5_AGENTIC_TRACK.md) | **Agent** frameworks (OpenAI SDK, CrewAI, LangGraph, AutoGen); **runtime** still deploys as **Python HTTP** (see **§2** here). |
| [`6_MLOPS_TRACK.md`](6_MLOPS_TRACK.md) | Week-by-week **project mapping**, Terraform, **reference repos** (split static + API vs **full platform**). |

---

## 9. References (external)

- [AWS App Runner — How it works](https://docs.aws.amazon.com/apprunner/latest/dg/what-is-apprunner.html) (official)
- [Amazon ECR](https://docs.aws.amazon.com/ecr/) (official)
- [AWS Lambda Web Adapter](https://github.com/awslabs/aws-lambda-web-adapter) (GitHub)
- [Lambda function URLs](https://docs.aws.amazon.com/lambda/latest/dg/urls-invocation.html) (official)
- [ai-financial-planner](https://github.com/aditya-caltechie/ai-financial-planner) — multi-service AWS example
- [Grok share — “AWS Deployment: No Frontend Knowledge Needed”](https://grok.com/share/bGVnYWN5_909a299a-13fd-469d-9e2b-a14c83a0f4b4?rid=a52f275d-638d-4b91-a599-5dbcc2d0b42a) — inspiration for backend-first framing
