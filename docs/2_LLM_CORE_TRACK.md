# LLM Engineering Core Track — Mental Map & Review

## High-level course summary

This course takes you from **calling frontier LLMs over APIs** to **running open models with Hugging Face Transformers**, then down two parallel tracks of **adaptation**: **inference-time** (RAG, prompting, multi-model workflows) and **weight-time** (baselines, neural models on text, frontier fine-tuning, QLoRA on open models). You close with a **Week 8 capstone** that looks like real products: **retrieval + API LLMs + deployed fine-tuned specialists + agents, memory, and ensembles**.

**In one sentence:** Learn to **engineer** LLM applications — not just prompt them — by combining APIs, local inference, RAG with evaluation, training and fine-tuning, and multi-agent hybrid systems.

**Core through-line:**

| Phase | What you practice |
|--------|-------------------|
| **Weeks 2–4** | Inference and composition: APIs, Gradio, HF `pipeline` / tokenizer / model, harder multi-model coding tasks. |
| **Week 5** | **RAG**: chunk, embed, vector DB (Chroma), LangChain retrieval, **measure** retrieval and answers. |
| **Weeks 6–7** | **Same business problem (price prediction)** from data curation → baselines → PyTorch → **fine-tuning** (frontier + QLoRA). |
| **Week 8** | **Integrate** RAG-style context, frontier pricing, Modal-hosted fine-tuned model, planning, and memory. |

**Ideas to keep in mind:** Transformers are the usual architecture under the hood; **Hugging Face** gives you both quick **`pipeline()`** inference and **lower-level** APIs that fine-tuning and custom generation need. **Product skills** (UIs, evals, deployment) sit beside model skills all the way through.

Refer to [NOTES](https://github.com/aditya-caltechie/ai-tutorial-notes/tree/main/llm-core) first for organised course overview.

---

### Course flow

```
                    ┌──────────────────────────────────--───┐
                    │  Week 1 (primer): chat + basic API    │
                    └──────────────────┬───────────────-────┘
                                       │
         ┌─────────────────────────────┴─────────────────────────────┐
         │                     WEEK 2: Product shell                 │
         │   Frontier APIs • Gradio • conversational apps            │
         └─────────────────────────────┬─────────────────────────────┘
                                       │
         ┌─────────────────────────────┴─────────────────────────────┐
         │              WEEK 3: HF / Transformers locally            │
         │   pipeline() • tokenizer • model • generate • quantize    │
         └─────────────────────────────┬─────────────────────────────┘
                                       │
         ┌─────────────────────────────┴─────────────────────────────┐
         │         WEEK 4: Harder inference (multi-model code)       │
         └─────────────────────────────┬─────────────────────────────┘
                                       │
                    ┌──────────────────┴──────────────────┐
                    │                                     │
                    ▼                                     ▼
    ┌───────────────────────────────┐     ┌───────────────────────────────┐
    │  WEEK 5: RAG(no weight change)│     │ WEEK 6–7: Adapt WEIGHTS       │
    │  chunk • embed • Chroma       │     │ data • baselines • DL • FT    │
    │  LangChain retriever + LLM    │     │ QLoRA / open-source FT        │
    │  eval: MRR/NDCG/answers       │     │ same “pricer” narrative       │
    └───────────────┬───────────────┘     └───────────────┬───────────-───┘
                    │                                     │
                    └──────────────────┬──────────────────┘
                                       ▼
                    ┌──────────────────────────────────────┐
                    │  WEEK 8: Multi-agent HYBRID system   │
                    │  RAG context + frontier LLM          │
                    │  + fine-tuned specialist (Modal)     │
                    │  + ensemble + planning + memory      │
                    └──────────────────────────────────────┘
```

---

### Mental map (three strands → capstone)

```
                         [ LLM ENGINEERING TRACK ]
                                    |
        +---------------------------+---------------------------+
        |                           |                           |
   [INFERENCE PATH]            [WEIGHT PATH]              [ENGINEERING]
        |                           |                           |
   Week2 APIs                 Week6–7 FT                 Week2 Gradio
   Week3 HF infer             QLoRA / PEFT               Week5 Eval UI
   Week4 multi-model          Datasets/trainers          Week8 Modal deploy
   Week5 RAG                  Frontier FT (W6)           Agents/orchestration
        |                           |                           |
        +-----------+---------------+---------------+-----------+
                    |               |               |
              [Prompting]     [RAG/Retrieval]  [Fine-tune]
                 Week2            Week5         Week6–7
                    \               |               /
                     \              |              /
                      \             |             /
                       v            v            v
                  ┌────────────────────────────────┐
                  │   Week 8: ENSEMBLE + PLANNING   │
                  │   Chroma + ST embeddings        │
                  │   FrontierAgent + SpecialistAgent│
                  └────────────────────────────────┘
```

---

**About this document:** The sections below walk **week by week** with pointers into the **notebooks and Python modules** in this repo. The same flow and mind map also appear later as **Sections 8–9** for reference while scrolling.

---

## 1. Frontier APIs and application shape (Week-2)

**Theme:** Talk to **multiple providers** (OpenAI and, in the course materials, others) through their APIs; wrap behavior in **Gradio**; build **conversational** apps.

**Representative ideas in repo:**

- API clients, keys via `.env`, simple chat patterns.
- **Gradio** for quick UIs (`week2/day2.ipynb`).
- **Conversational AI / chatbot** patterns (`week2/day3.ipynb`).
- Broader Week 2 labs extend into tools, richer agents, and multimodal experiments (including community notebooks).

**Mental model:** This week is about **inference-time** use of powerful models: you are not training; you are **composing** calls, state, and UX.

---

## 2. Hugging Face, pipelines, and the “local model” stack (Week-3)

**Theme:** Move from “API-only” to **running models with Transformers** (often in Colab/GPU), understanding **tokenizers** and **model objects**, and using **high-level `pipeline()`** where it fits.

**Day titles in core notebooks:**


| Day | Focus (from notebook headings)                                  |
| --- | --------------------------------------------------------------- |
| 1   | Introducing Colab                                               |
| 2   | Hugging Face **pipelines**                                      |
| 3   | **Tokenizers**                                                  |
| 4   | **Models**                                                      |
| 5   | **Meeting minutes** (multimodal path: audio → transcript → LLM) |


**Libraries and patterns you actually touch:**

- `transformers`: `pipeline`, `AutoTokenizer`, `AutoModelForCausalLM`, `generate`, chat templates, streaming (`TextStreamer`).
- **Quantization** for feasible local inference: `BitsAndBytesConfig` (4-bit / NF4 patterns appear in course and community examples).
- **Hugging Face Hub** auth (`huggingface_hub.login`, tokens).
- Optional bridges to **frontier APIs** alongside HF models (dataset generation and tooling examples in community contributions).

**Clarifying the “two levels” of HF:**


| Level                           | Typical use                                                 | In this track                                                              |
| ------------------------------- | ----------------------------------------------------------- | -------------------------------------------------------------------------- |
| `**pipeline(task, model=...)`** | Fast baseline for standard tasks (summarization, ASR, etc.) | Week 3 pipelines; Whisper-style ASR shows up in meeting-minutes style labs |
| **Manual model + tokenizer**    | Control inputs, chat templates, batching, fine-tuning hooks | Week 3 models/tokenizers; essential for Week 6–7 training workflows        |


**Not identical but related:** `huggingface_hub` (download, auth), `datasets`, PEFT, and trainers — these become central when you **fine-tune** (Weeks 6–7).

---

## 3. Multi-model synthesis: “code intelligence” week (in this repo) (Week-4)

**Note:** This checkout includes `**week4/day3.ipynb` through `day5.ipynb`**. The theme there is **using several frontier models** for demanding **code generation / translation** tasks (e.g. Python → high-performance C++ with compile/run checks in the lab narrative).

**Mental model:** Week 4 reinforces **inference-only orchestration** at a harder skill ceiling: model selection, iterative prompting, and validating outputs — still **not** training your own LLM.

---

## 4. RAG: retrieval as the main inference-time adaptation (Week-5)

**Theme:** Build an **expert knowledge worker** for a fictional company (Insurellm) using **RAG** — accurate, grounded answers with **lower reliance on parametric memory**.

**Progression (matches `week5/day1`–`day5` and `implementation/`):**

1. **Naive / brute-force baseline** — load knowledge into memory, answer with an LLM (`day1`).
2. **Chunking + embeddings + vector store** — `RecursiveCharacterTextSplitter`, **Chroma**, OpenAI or HF embeddings (`day2`).
3. **LangChain 1.0 RAG chain** — retriever + chat model + Gradio UI (`day3`).
4. **Evaluation** — test sets, retrieval quality, answer scoring; Gradio evaluator (`day4`, `evaluation/`, `evaluator.py`).
5. **Advanced / “Pro” RAG** — LLM-assisted chunking and **document preprocessing** (headlines/summaries), optional **native Chroma** path for flexibility (`day5`, `pro_implementation/`, `implementation/`).

**Inference vs fine-tuning:** All of Week 5 is **inference-time adaptation**: you change **what context the model sees**, not its weights.

**Techniques to name explicitly (you implied RAG; the repo also implements):**

- **Chunking strategy** and overlap (recall vs precision tradeoff).
- **Embedding model choice** (e.g. OpenAI `text-embedding-3-large` vs `sentence-transformers/all-MiniLM-L6-v2`).
- **Retrieval metrics**: MRR, NDCG, coverage-style signals (`week5/evaluation/eval.py`, `evaluator.py`).
- **Answer quality** rubrics (accuracy, relevance, completeness scales in evaluator UI).
- **Agentic RAG** appears in **community contributions** (tool loops: vector search, rerank, judge, final answer) — worth knowing as the “next step” beyond a single retriever call.

---

## 5. Capstone part A: data, baselines, classical ML, small/deep nets, frontier fine-tune (Week-6)

**Project:** **“The Price Is Right”** — predict a **price** from a **product description** (Amazon-scrape style data).

**Order of play (from `week6/day1` / `day2` intros):**


| Day | Emphasis                                                                                                              |
| --- | --------------------------------------------------------------------------------------------------------------------- |
| 1   | **Data curation** — filtering, distributions, train/val thinking                                                      |
| 2   | **LLM-based preprocessing** — rewrite to a standard format (ties back to Week 5 “LLM rewrites text”)                  |
| 3   | **Evaluation**, **baselines**, **traditional ML** (e.g. linear regression, random forest on engineered text features) |
| 4   | **Deep learning** on text features + **LLM-in-the-loop** pricing approaches                                           |
| 5   | **Fine-tuning a frontier model** for the same task                                                                    |


**Code anchors:**

- `week6/pricer/` — items, batches, training utilities; `deep_neural_network.py` implements a serious **PyTorch** stack (residual MLP-style blocks) for price regression from hashed text features.

**Mental model:** Week 6 is the **full modeling ladder**: from **cheap baselines** to **neural** predictors to **LLM fine-tuning**, with **evaluation** tying them together. That ladder is what makes Week 7–8 comparisons meaningful.

---

## 6. Capstone part B: open-source LLM fine-tuning -QLoRA track (Week-7) 

**Theme:** Fine-tune an **open-weight** model for the pricer task using **parameter-efficient** methods.

**From `week7/day1.ipynb`:** QLoRA, prompt/data prep, train (parts 1–2), eval — with a **Colab** notebook link for GPU training.

**Mental model:** Week 7 is **weight adaptation** with **efficiency**: you are not necessarily full fine-tuning; you are using **LoRA adapters** and **quantized** base models (QLoRA) to fit consumer/colab GPUs.

---

## 7. Multi-agent systems: **hybrid production-shaped stack** (Week-8)

**Theme:** **Autonomous deal-finding / pricing** agents that combine:

- **RAG-style retrieval** over a **Chroma** collection of products (`products_vectorstore`).
- **Frontier LLM** pricing with **similar-product context** (`week8/agents/frontier_agent.py` — embed query, retrieve neighbors, prompt GPT).
- **Specialist** agent calling a **Modal-hosted fine-tuned pricer** (`week8/agents/specialist_agent.py` + `pricer_service*.py`).
- **Ensemble** blending specialist + frontier outputs (`week8/agents/ensemble_agent.py` — weighted combination after shared preprocessing).
- **Planning / autonomy** (`deal_agent_framework.py` — `AutonomousPlanningAgent`, persistent `memory.json`).
- **Optional depth**: neural network agent hook exists in ensemble code path but may be commented/disabled in the snapshot you have — the **pattern** is still part of the course design (stacking heterogeneous predictors).

**Mental model:** Week 8 is the **integration week**: the same business problem (price/deals) now forces you to reason about **routing**, **memory**, **remote inference**, and **heterogeneous models** — the practical shape of many real GenAI products.

---

## 8. What you learned and can now review (checklist)

- **Call frontier models** responsibly from code (keys, env, model choice, cost).
- **Ship thin UIs** around models with **Gradio**.
- **Run open models** with **Transformers**: pipelines for speed, manual APIs for control.
- **Understand tokens** and why they matter for cost, context, and training.
- **Build RAG**: chunking, embeddings, **Chroma**, **LangChain 1.0** patterns, and **evaluation** beyond “vibes.”
- **Improve RAG** with LLM-based chunking and **document preprocessing** (Pro track).
- **Curate and preprocess** domain data with LLMs (Week 6 — parallel to Week 5 advanced ingest philosophy).
- **Measure models** with baselines, classical ML, **PyTorch** models, and **LLM fine-tuning**.
- **Fine-tune open models** with **QLoRA** (Week 7).
- **Integrate** retrieval, frontier inference, fine-tuned specialists, and **agentic orchestration** (Week 8).

---

## 9. Optional “next layer” not fully spelled out in core weekly folders

These show up in **community contributions**, partial hooks, or adjacent files — useful for self-study:

- **Agentic RAG** with explicit tool policies and multi-step retrieval.
- **LiteLLM** as a unified completion layer (appears in some Week 5 advanced paths).
- **xgboost / random forest** and other tabular-style stacks on engineered features (Week 6 trajectory; some Week 8 community ensembles).
- **Production hardening**: logging, idempotent ingest, CI tests under `week8/community_contributions/`.

---

## 12. Key file pointers (for quick navigation)


| Area                               | Location                                                      |
| ---------------------------------- | ------------------------------------------------------------- |
| RAG answer + retriever (LangChain) | `week5/implementation/answer.py`                              |
| RAG evaluation                     | `week5/evaluation/`, `week5/evaluator.py`                     |
| Advanced ingest                    | `week5/pro_implementation/`, `week5/day5.ipynb`               |
| Pricer DNN                         | `week6/pricer/deep_neural_network.py`                         |
| Agent framework                    | `week8/deal_agent_framework.py`                               |
| RAG-style pricing + frontier LLM   | `week8/agents/frontier_agent.py`                              |
| Remote fine-tuned pricer           | `week8/agents/specialist_agent.py`, `week8/pricer_service.py` |
| Ensemble                           | `week8/agents/ensemble_agent.py`                              |


---

*Generated as a course review map from the repository layout and core notebooks in `week2`–`week8`. If your local branch differs (e.g. extra Week 4 days), merge those notebooks into section 3 mentally.*
