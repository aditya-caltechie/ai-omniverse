# ai-omniverse

**One stop for AI/ML learning** — AI/ML learning gym: traditional ML, deep learning, fine-tuning, Gen AI, agentic AI. Everything from experiments to production with evaluation and observability.

This repo is a curated learning path covering the full spectrum of modern AI and machine learning:


| **Area** | **Coverage** |
| -------- | ------------ |
| **Traditional ML**    | Fundamentals: regression, classification, clustering, feature engineering, and model evaluation. [Github](https://github.com/aditya-caltechie/handson-mlp)             |
| **Deep Learning**     | Neural networks, CNNs, RNNs, transformers, and training at scale.                                                                                                      |
| **Gen AI & LLMs**     | Building applications with large language models: prompting, RAG, advance RAG, agentic RAG, and deployment.                                                            |
| **Fine Tuning (SFT)** | Fine-tuning small language models ( PEFT e.g. LoRA, QLoRA, adapters) on custom data. You can perform SFT using Full Fine-Tuning (expensive) or LoRA/QLoRA (efficient). |
| **RLHF Pipeline**     | RLHF pipeline (SFT → Reward model → PPO).                                                                                                                              |
| **Agentic AI**        | Autonomous agents, tool use, multi-step reasoning, and agent frameworks.                                                                                               |
| **MLOps**             | Experiment tracking, model versioning, pipelines, serving, and monitoring in production, along with EVALs and observability.                                           |


---

## AI roadmap & hierarchy - mind map

```mermaid
mindmap
  root((AI mastery))
    Foundations
      Classical ML
      Python stats linear algebra
    Deep learning
      NN from scratch
      CNNs RNNs
    Transformers and LLMs
      Architecture attention
      Base model checkpoint
    Inference time
      APIs prompting
      RAG evals
    Weight time
      Continued pretrain
      SFT full FT
      LoRA QLoRA
    Alignment optional
      DPO RLHF preferences
    Agentic systems
      Tools loops memory
      Frameworks MCP
    Production
      MLOps deploy
      Observability guardrails cost
```

### Logical flow — dependency ladder

```
                         ┌─────────────────────────────┐
                         │  SHIP  ·  MLOps · evals     │
                         │  guardrails · cost · SLOs   │
                         └──────────────┬──────────────┘
                                        │
                         ┌──────────────▼──────────────┐
                         │  AGENTS  · tools · loops    │
                         │  crews · graphs · MCP       │  ← docs/5_AGENTIC_TRACK.md
                         └──────────────┬──────────────┘
                                        │
              ┌─────────────────────────┴───────────--──────────────┐
              │                                                     │
   ┌──────────▼──────────┐                               ┌──────────▼──────────┐
   │ INFERENCE-TIME      │                               │ WEIGHT-TIME         │
   │ prompts · RAG · API │                               │ CPT · SFT · FT      │
   │ (no weight change)  │                               │ LoRA · QLoRA        │
   └──────────┬──────────┘                               └──────────┬──────────┘
              │                                                     │
              └───────────────────────────┬─────────────────────────┘
                                          │
                         ┌────────────────▼────────────────┐
                         │  TRANSFORMERS / LLMs            │
                         │  attention · tokenizer · base   │
                         │  HF · generate · chat templates │  ← docs/2_LLM_CORE_TRACK.md
                         └────────────────┬────────────────┘
                                          │
                         ┌────────────────▼────────────────┐
                         │  DEEP LEARNING                  │
                         │  backprop · CNN · RNN           │  ← ai-deep-learning
                         └────────────────┬────────────────┘
                                          │
                         ┌────────────────▼────────────────┐
                         │  FOUNDATIONS                    │
                         │  classical ML (sup/unsup)       │
                         └─────────────────────────────────┘

  Sidecar ideas (plug in when needed): alignment (DPO / RLHF) · multimodal ·
  synthetic data · RAG eval (MRR, faithfulness) · red-teaming.
  Training *stages* cheat sheet: docs/3_LLM-TRAINING.md · adaptation methods: docs/4_FINE-TUNING.md
```

---

## 1. AI/ML Foundation


| **Topic** | **Repo** | **Description** |
| --------- | -------- | --------------- |
| Traditional ML (**optional**) | Todo                                                                     | Todo                                                                                                                                                                                                                                    |
| Deep Learning (**optional**)  | [ai-deep-learning](https://github.com/aditya-caltechie/ai-deep-learning) | Building your own NN from scratch. It also has workshop which covers - Deep learning fundamentals with PyTorch/TensorFlow: neural networks, CNNs, RNNs, and training pipelines. Foundation for understanding how modern LLMs are built. |


## 2. LLM Core Track

**Transformers** are the modern day hero. All LLM are based on transformer architecture. 
Besides that, there are two major concept : 

1. **Inference techniques** ( Prompt Engg, RAG, Agentic-RAG ).
2. **Training techniques**  ( Fine-tuning - Full-fine-tuning / PEFT(LORA/QLORA )).

**In-repo guides:**  [LLM_CORE_TRACK](docs/2_LLM_CORE_TRACK.md)  |  [LLM-TRAINING](docs/3_LLM-TRAINING.md)  |  [FINE-TUNING](docs/4_FINE-TUNING.md)


| **Topic** | **Repo** | **Description** |
| --------- | -------- | --------------- |
| Transformers                              | [ai-transformers](https://github.com/aditya-caltechie/ai-transformers)         | All about Transformers. High-level APIs for Inference. Low-level APIs for fine-tuning [Illustration](https://jalammar.github.io/illustrated-transformer/)                                                                                                      |
| Pre-Training LLM (**optional**)           | [ai-pretraining](https://github.com/aditya-caltechie/ai-pretraining/tree/main) | Continous Pre-training. why and when its useful and needed                                                                                                                                                                                                     |
| Fine-tuning (SFT) - PEFT/LORA             | [ai-fine-tuning](https://github.com/aditya-caltechie/ai-fine-tuning)           | Fine-tuning small language models (e.g. LoRA, QLoRA, adapters) on custom data. Covers data prep, training, and evaluation for domain-specific models. This also has workshop section from March28. Material from DeepLearning.ai course as well for reference. |
| RLHF pipeline (**optional**)              | [Course](https://huggingface.co/learn)                                         | Todo                                                                                                                                                                                                                                                           |
| LangChain Basics                          | [ai-langchain-intro](https://github.com/aditya-caltechie/ai-langchain-intro)   | Introduction to LangChain: chains, prompts, output parsers, and connecting to LLMs. Covers the core building blocks for LLM applications.                                                                                                                      |
| RAG                                       | [ai-rag](https://github.com/aditya-caltechie/ai-rag)                           | Retrieval-augmented generation: vector stores, embeddings, and chaining retrievers with LLMs for knowledge-grounded answers.                                                                                                                                   |
| Agentic RAG                               | [ai-agentic-rag](https://github.com/aditya-caltechie/ai-agentic-rag)           | Agentic RAG: agents that decide when and how to search and reason over retrieved context using LangChain/LangGraph-style patterns.                                                                                                                             |
| Multi Agents using Tools/loop & Workflows | [ai-deals2buy](https://github.com/aditya-caltechie/ai-deals2buy)               | Capstone: AI agent for deals/shopping with LLM calls, tools, and agent logic. Two approaches: (a) loops & tools via `autonomous_planning_agent.py`, (b) workflow via `planning_agent.py`.                                                                      |


---

## 3. Agentic Core Track

**In-repo guide:** **[5_AGENTIC_TRACK.md](docs/5_AGENTIC_TRACK.md)** (framework comparison, Mermaid diagrams, MCP vs orchestration)

| **Topic** | **Repo** | **Description** |
| --------- | -------- | --------------- |
| Agents without Framework     | [ai-career-agent](https://github.com/aditya-caltechie/ai-career-agent)                           | Basic agent without using any framework. Simple use of tools in loop with LLM to build career chat                                       |
| OpenAI Agents SDK            | [ai-agentic-sales-outreach](https://github.com/aditya-caltechie/ai-agentic-sales-outreach)       | Cold-send agent project: agentic cold sales email with tools and handoffs strategies. Uses worksflows and agent approach. Use guardrails |
| OpenAI Agents SDK            | [ai-openai-sdk-deep-research](https://github.com/aditya-caltechie/ai-agent-sdk-deep-research)    | Perform Deep Research                                                                                                                    |
| MCP (Model Context Protocol) | [ai-mcp-autonomous-traders](https://github.com/aditya-caltechie/ai-mcp-autonomous-traders)       | Autonomous trading agents using MCP: agents that use tools and context via the Model Context Protocol for trading workflows.             |
| CrewAI Framework             | [ai-crew-engineering-team](https://github.com/aditya-caltechie/ai-crew-engineering-team)         | Multi-agent "engineering team" with CrewAI: role-based agents collaborating on tasks. Demonstrates CrewAI's agent and task APIs.         |
| CrewAI Framework             | [ai-crew-financial-researcher](https://github.com/aditya-caltechie/ai-crew-financial-researcher) | Financial research agent built with CrewAI: research tasks, tools, and structured outputs for finance use cases.                         |
| CrewAI Framework             | [ai-crew-stock-picker](https://github.com/aditya-caltechie/ai-crew-stock-picker)                 | Stock-picking agent with CrewAI: agents that analyze and recommend stocks using external data and tools.                                 |
| LangGraph                    | [ai-sidekick](https://github.com/aditya-caltechie/ai-sidekick)                                   | Personal Assistance, that backs you up and helps you get things done                                                                     |
| Autogen AgentChat            | [ai-autogen-itinerary-planner](https://github.com/aditya-caltechie/ai-autogen-itinerary-planner) | Multi-agent workflow, where we can plan flights with optimal costs                                                                       |
| Autogen Core                 | [ai-autogen-core-judge](https://github.com/aditya-caltechie/ai-autogen-core-judge)               | Multi-agent workflow, where Judge decides results of two research agents                                                                 |
| Autogen Core                 | [ai-agent-creator](https://github.com/aditya-caltechie/ai-agent-creator)                         | Creates agents with buisiness ideas                                                                                                      |


---

## 4. MLOps Track


| **Topic** | **Repo** | **Description** |
| --------- | -------- | --------------- |
| Deploy Gen AI & Agentic AI in Production | [MLOps-production](https://github.com/aditya-caltechie/ed-donner-MLOps-production) | Course repo: deploy Gen AI and Agentic AI at scale in 4 weeks. Covers production deployment, guides, and week-by-week materials. |


---

## Study Notes


| **Repo** | **Description** |
| -------- | --------------- |
| [ai-tutorial-notes](https://github.com/aditya-caltechie/ai-tutorial-notes) | Consolidated notes and references from Udemy AI/ML courses. Quick lookup for concepts, commands, and patterns used across the tracks. |


---

## Track guides and mental maps

These files live in **`docs/`** (repository root). They **thread the same topics** as the repo tracks and the **Appendix** further down this README, but as **guided narratives** (week arcs, training stages, fine-tuning choices, agent stacks). Open them when you want diagrams and FAQs, not only quick definitions.


| Doc | What it covers | Theory in one line |
| --- | -------------- | ------------------ |
| **[1_ROADMAP.md](docs/1_ROADMAP.md)** | **Course / topic roadmap** (notebooks, resources) | Optional **fundamentals** vs **LLM scientist** vs **LLM engineer** paths; external course structure and handbook links—use as a bird’s-eye supplement to this repo’s tracks. |
| **[2_LLM_CORE_TRACK.md](docs/2_LLM_CORE_TRACK.md)** | Full **LLM engineering** arc (weeks, capstone) | **Inference-time** path (APIs → Hugging Face → RAG + eval) runs in parallel with **weight-time** adaptation (data → baselines → fine-tuning / QLoRA); **Week 8** hybrid combines RAG, frontier models, fine-tuned specialists, and agents. |
| **[3_LLM-TRAINING.md](docs/3_LLM-TRAINING.md)** | **Pre-training vs fine-tuning** vs education | **Foundation pre-training** builds the base checkpoint; **continued pre-training** nudges it on more unlabeled domain text; **full FT** and **PEFT** adapt on task data; **`ai-deep-learning`** teaches architecture + small-data training—not web-scale foundation PT. |
| **[4_FINE-TUNING.md](docs/4_FINE-TUNING.md)** | **SFT** and parameter-efficient methods | **SFT** teaches behavior from labeled pairs; **full FT** updates all weights; **LoRA / QLoRA** train small adapters; **prompt / prefix tuning** steers the model with few trainable parameters. |
| **[5_AGENTIC_TRACK.md](docs/5_AGENTIC_TRACK.md)** | **Agent frameworks** + MCP | Shared base: **LLM + tools + multi-step control**; compare **OpenAI Agents SDK**, **CrewAI**, **LangGraph**, **AutoGen** by orchestration style; **MCP** is a **tool/resource protocol**, not a replacement for those orchestrators. |


### How the guides connect (ASCII mind map)

```
                    [ 1_ROADMAP.md — optional bird's-eye ]
                                    |
                                    v
                         [ 2_LLM_CORE_TRACK.md ]
                            full curriculum arc
                                     |
         +-------------+-------------+----------------+--------------+
         |             |                              |              |
         v             v                              v              v
  3_LLM-TRAINING.md   4_FINE-TUNING.md   RAG (inference-time)     AGENTIC_TRACK.md
  foundation CPT FT   SFT ·full FT/PEFT  chunk ·embed ·vectorDB   handoffs · crews
  · FT · PEFT         · LoRA · prompts   · retrieve + LLM ·       · graphs · teams
  · scratch NN                           · agentic RAG · evals    · MCP (see guide)
                                         · repos: ai-rag 
         |             |                          |                  |
         +-------------+--------------------------+------------------+
                                 |
                                 v
              Product: grounded answers (RAG) + adapted weights + agent orchestration
              (Week 8 / capstone style — see docs/2_LLM_CORE_TRACK.md)
```

---

# Appendix :

### Frameworks:

#### OpenAI Agents SDK, CrewAI, and AutoGen

These are all **true agent frameworks** (or orchestration frameworks). They're built specifically to let you quickly spin up AI agents (or teams of agents) with opinionated patterns:

- OpenAI Agents SDK → lightweight, production-ready primitives for single/multi-agent orchestration (successor vibe to Swarm).
- CrewAI → role-based "crews" where agents have jobs, tasks, and handoffs like a human team.
- AutoGen → conversation-based multi-agent chats where agents talk to each other to solve problems.

They're designed end-to-end for "build an agent/team fast."
LangChain / LangGraph sit in a different bucket.

#### LangChain / LangGraph sit in a different bucket.

LangChain is a general LLM application **framework** (chains, memory, RAG, tools, etc.). LangGraph is its stateful graph layer for building custom workflows with cycles, branching, and multi-actor control flow. You can build agents with them (tons of people do), but they're lower-level building blocks rather than ready-made "agent frameworks." You have to design most of the orchestration yourself — it's more like "React for agents" than "a crew/team builder." 

LangGraph is very much a framework, but it's a different kind compared to things like CrewAI, AutoGen, or OpenAI Agents SDK — which is why it can feel like it doesn't "fit" the same bucket.

The key distinction comes down to level of abstraction and opinionation:

- Higher-level / opinionated agent frameworks (CrewAI, AutoGen, OpenAI Agents SDK, etc.)
These give you ready-made patterns: "agents with roles," "conversations between agents," "handoffs/swarm-style delegation," predefined task queues, automatic tool routing, etc. You mostly configure agents/roles/tasks, and the framework handles a lot of the orchestration "magic" for you. Great for speed → build a team of agents in ~100 lines.
- LangGraph (and to some extent LangChain itself)
It's a **low-level orchestration framework** / agent runtime built around explicit, programmable graphs (directed graphs with nodes, edges, conditional branches, cycles/loops, persistence). LangGraph is a framework — specifically a low-level, graph-based agent orchestration framework/runtime for stateful, controllable, production-grade AI agents and workflows.

#### LangGraph can we used for making both :

- **Workflows** : example traditional rag. Simpler. [Reference-1](https://github.com/aditya-caltechie/ai-langchain-intro/blob/main/docs/LangGraph.md) | 
[Reference-2](https://github.com/aditya-caltechie/ai-langchain-intro/tree/main/src/rag)
- **Full Agnetic solution** : example agentic-rag repo, ai-sidekick repo

#### MCP (Model Context Protocol) :

It is even further removed — it's not a framework at all. It's a protocol/standard. It just defines a clean, standardized way for any agent to discover and call tools/context/resources from MCP servers.

---

### Fine-Tuning Terminology (SFT vs PEFT/LoRA/QLORA vs Full FT)


| **Term** | **Category** | **What it describes** |
| -------- | ------------ | --------------------- |
| SFT                             | Learning Paradigm | Using labeled *input/output* data to teach a specific behavior.                                                                                                                                                                                                                                                                                                                                                                                           |
| Full Fine-Tuning                | Resource Method   | Modifying **100%** of the model's weights during SFT. [example-1](https://github.com/aditya-caltechie/ai-deep-learning/blob/main/src/workshop/text_classification.py), [example-2](https://github.com/aditya-caltechie/ai-deep-learning/blob/main/src/workshop/question_answering.py)                                                                                                                                                                     |
| PEFT Fine-Tuning (LoRA / QLoRA) | Resource Method   | Modifying **< 1%** of the model's weights during SFT (parameter-efficient). [example-1](https://github.com/aditya-caltechie/ai-deep-learning/blob/main/src/workshop/fine_tuning.py), [example-2](https://github.com/aditya-caltechie/ai-fine-tuning/blob/main/src/fine_tuning/notebooks/2_fine-tuning_via_QLORA.ipynb), [example-3](https://github.com/aditya-caltechie/ai-fine-tuning/blob/main/src/workshop/Finetuning_Workshop_SFT_Demo_March28.ipynb) |


### Stages vs. What They Learn


| **Stage** | **What is learned?** | **Analogy** |
| --------- | -------------------- | ----------- |
| SFT        | Domain knowledge & format   | Learning the textbook for a class.         |
| DPO / RLHF | Style, safety, & preference | Taking practice exams and getting a grade. |


### Two-Stage Adaptation Pattern


| **Stage** | **Goal** | **Typical method** | **Notes** |
| --------- | -------- | ------------------ | --------- |
| `Stage 1` – Task & domain (SFT)    | Teach domain knowledge, task behavior, and output format. | Supervised Fine-Tuning (SFT), usually with **PEFT (LoRA / QLoRA)** instead of full fine-tuning. | Parameter‑efficient; far less compute and memory than full FT, and often sufficient to ship a product. |
| `Stage 2` – Alignment & refinement | Refine style, helpfulness, safety, and response quality.  | Preference-based alignment via **DPO** or **RLHF** on preference data.                          | Optional second pass; most valuable when UX and safety need additional optimization.                   |


#### Flow with Methods (Where each fits)

```
              ┌────────────────────┐
              │   Pretraining      │
              │ (Huge data, $$$)   │
              └────────┬───────────┘
                       ↓
              ┌────────────────────┐
              │  Base Model        │
              └────────┬───────────┘
                       ↓
        ┌────────────────────────────────┐
        │  Supervised Fine-Tuning (SFT)  │
        │  - Instruction tuning          │
        │  - Domain tuning               │
        └────────┬───────────────────-───┘
                 ↓
        ┌────────────────────────────────┐
        │  HOW to fine-tune (Technique)  │
        │  - Full FT                     │
        │  - PEFT (LoRA, QLoRA)          │
        └────────┬────────────────────-──┘
                 ↓
        ┌────────────────────────────────┐
        │ Alignment Stage (Post Training)│
        │  - RLHF (uses PPO)             │
        │  - DPO                         │
        │  - RL (PPO etc.)               │
        └────────┬────────────────────-──┘
                 ↓
          ┌────────────────────┐
          │ Final Assistant    │
          └────────────────────┘
```

#### End to end pipeline:

```
1. Pick base model (LLaMA / Mistral)
2. Prepare dataset (instruction format)
3. Run Fine-tuning: SFT + LoRA  
4. (Optional) Run DPO for alignment
5. Save + merge adapters
6. Serve using vLLM
7. Build app using LangChain
```

**BEST PRACTICAL STACK**

```
HuggingFace Transformers → loads model  (Model: Mistral / LLaMA )
        ↓
PEFT (LoRA) → makes tuning cheap
        ↓
Trainer / TRL → runs SFT / DPO
        ↓
vLLM → serves model fast
        ↓
LangChain → builds app/agent
```

There are 3 main steps:

- PreTraining - (Mostly done by Giants like Google, Meta)
- Fine-Tuning (SFT- FT/PEFT for domain training)
- Reinforcement Learning (RLHF/DPO) - Optional

```
👉 SFT teaches the model WHAT to say
👉 PEFT makes it CHEAP to train
👉 DPO/RLHF teaches HOW to behave
```

