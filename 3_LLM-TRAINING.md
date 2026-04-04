# LLM training stages — course map

This note ties together **foundation pre-training**, **continued pre-training**, **fine-tuning**, **PEFT**, and **from-scratch NN** literacy. Read top to bottom, or jump: [examples](#real-world-examples-2026) → [ASCII map](#conceptual-map-ascii) → [at-a-glance table](#at-a-glance) → [detailed blocks](#pre-training) → [FAQ](#faq-pre-training-vs-building-a-nn-vs-continued-pre-training) → [repo learning path](#learning-path-your-repos).

---

## Real-world examples (2026)

- **Pre-training:** xAI training Grok-class models from scratch on massive unlabeled data (foundation run).
- **Continued pre-training:** Upstage-style work — take a base model, train more on Korean + code → **TinySolar** (domain-heavy **unlabeled** continuation, not a new random-init foundation).
- **Full fine-tuning:** Take a base (e.g. TinySolar) and update **all** weights on thousands of instruction examples → chatbot that matches your style.
- **PEFT (LoRA):** Same adaptation goal, but only adapter weights train → much faster / cheaper (e.g. ~30 minutes on one A100 vs days for full FT).

---

## Bottom line for your repos

- **`ai-pretraining`** → **Continued pre-training** (more unlabeled domain data; **all** base weights can move; still starts from an **existing** checkpoint).
- **`ai-transformers/Week-3/full-fine-tuning`** → **Full fine-tuning** (task / labeled-style data; **all** weights updated).
- **`ai-transformers/Week-3/peft-fine-tuning`** → **PEFT** (LoRA / QLoRA; **most** base weights frozen).

All of the above assume **someone already produced a base** via foundation pre-training (you are not redoing random-init web-scale training in these repos).

---

## Conceptual map (ASCII)

**Foundation vs continuation vs task adaptation** (one screen-friendly view):

```
  TRUE (FOUNDATION) PRE-TRAINING          ← big labs (Meta, Google, xAI, …)
  ┌────────────────────────────────────┐
  │ NN architecture + random/near-random│
  │ init + HUGE unlabeled text          │
  │ (billions → trillions of tokens)    │
  └─────────────────┬──────────────────┘
                    │
                    ▼
            ┌───────────────┐
            │  BASE MODEL   │  e.g. BERT, Llama 3 (you usually DOWNLOAD this)
            └───────┬───────┘
                    │
    ┌───────────────┼───────────────────────────────┐
    │               │                               │
    ▼               │                               │
┌───────────────┐   │   ┌───────────────────────────────────────────┐
│ CONTINUED     │   │   │ TASK ADAPTATION (starts from BASE MODEL)    │
│ PRE-TRAINING  │   │   │ data: labeled / instructions / JSONL, etc.  │
│ more UNLABELED│   │   └───────────────┬───────────────────────────┘
│ domain text   │   │                   │
└───────┬───────┘   │           ┌─────────┴─────────┐
        │           │           ▼                   ▼
        ▼           │    FULL FINE-TUNING      PEFT (LoRA / QLoRA)
 e.g. TinySolar     │    all weights change    adapters only (~0.1–5%)
 ai-pretraining     │    Week-3/full-…         Week-3/peft-…
        │           │
        └───────────┘  (both branches still “after” a base exists)


  PARALLEL TRACK (education, not web-scale foundation):
  deep_neural_network.py  →  learn architecture + forward + training loop
                             on SMALL data (≠ “true” foundation pre-training)
```

**Mind-map style (same ideas, radial):**

```
                         [ BASE CHECKPOINT ]
                        /         |         \
                       /          |          \
                      /           |           \
           CONTINUED PT      FULL FT         PEFT
           unlabeled         all weights     adapters
           domain           labeled/task    cheap / fast
              |                 |                |
         TinySolar          IMDb, SQuAD      same tasks,
         ai-pretraining     full-fine-tun…   peft-fine-tun…


   FOUNDATION PT (upstream of everything above, rare in courses)
        │
        └── random init + massive corpus → creates BASE CHECKPOINT


   FROM-SCRATCH NN (orthogonal skill)
        └── layers, tensors, backprop — deep_neural_network.py
```

---

## Clarification

Wide multi-column Markdown tables often force **horizontal scrolling**. Below: a **narrow summary table**, then **stacked detail blocks** (full width, same content as a big table would hold).

### At a glance

| Style | Builds NN from scratch? | Updates all weights? | Typical compute |
| ----- | ----------------------- | -------------------- | --------------- |
| Pre-training | Yes | Yes | Very high (1000s of GPUs) |
| Continued pre-training | No | Yes | Medium (few GPUs) |
| Full fine-tuning | No | Yes | High (multi-GPU) |
| PEFT (LoRA / QLoRA) | No | No (~0.1–5% params) | Low (1 GPU / Colab OK) |
| From-scratch NN | Yes | Yes | Low–medium |

### Pre-training

- **What it is:** Massive first training phase to build a foundational/base model from (near-)random weights.
- **Data:** Huge unlabeled text (trillions of tokens).
- **Your repo:** None (labs like Meta, xAI, Google-scale).
- **Example models:** Llama 3, GPT-4–class, Gemini, Grok — large foundation LMs.
- **Typical use cases:** General chat, code, reasoning, multimodal bases; starting point for later full FT, PEFT, or distillation.

### Continued pre-training

- **What it is:** Start from an existing base and train further on new domain-heavy **unlabeled** text.
- **Data:** More unlabeled text (e.g. Python, Korean, legal, biomedical corpora).
- **Your repo:** `ai-pretraining` (TinySolar-248M + Python scripts).
- **Example models:** TinySolar-248M (Korean + code), CodeLlama-style extensions of Llama, other “second stage” checkpoints.
- **Typical use cases:** Shift vocabulary and style toward finance, law, medicine, multilingual, or code **before** task-specific fine-tuning.

### Full fine-tuning (full training)

- **What it is:** Pre-trained model + **every weight** updated on task-specific (often labeled) data.
- **Data:** Smaller labeled sets — classification, QA, instructions, etc.
- **Your repo:** `ai-transformers/Week-3/full-fine-tuning` (BERT on IMDb & SQuAD).
- **Example models:** BERT-base in your labs; Llama / Mistral fully updated on instruction or domain JSONL.
- **Typical use cases:** Sentiment, extractive QA, NER, custom instruction chatbots when you can afford full-weight training.

### PEFT (LoRA, QLoRA, …)

- **What it is:** Pre-trained model + **small adapter** weights; base mostly frozen.
- **Data:** Same kinds as full fine-tuning (labeled / instruction data).
- **Your repo:** `ai-transformers/Week-3/peft-fine-tuning` (LoRA / QLoRA examples).
- **Example models:** Llama 3 8B, Mistral 7B, Qwen2.5 — adapters on top of the same families you’d full FT.
- **Typical use cases:** Fast iteration, low VRAM, many small vertical adapters (support triage, tone, tagging) from one frozen base.

### From-scratch neural network

- **What it is:** You define the full architecture (layers, shapes) yourself — educational or small-scale training.
- **Data:** Any; usually a **small** dataset for learning or demos.
- **Your repo:** `ai-deep-learning/src/core/deep_neural_network.py`.
- **Example models:** Custom MLPs; small CNNs on MNIST-style data in tutorials.
- **Typical use cases:** Teaching backprop and layer design; baselines and ablations; tiny embedded or specialized models where a pretrained LM is not the goal.

---

## FAQ: pre-training vs building a NN vs continued pre-training

### 1. Is “pre-training” the same as building your own NN from scratch?

**No — they overlap, but they are not the same thing.**

- **Build NN from scratch** — You **define the architecture** (layers, heads, embeddings, …) with `torch.nn.Module`. Example: `deep_neural_network.py` — *structure and forward pass*, not necessarily web-scale training.

- **True foundation pre-training** — **Architecture + (usually fresh) initialization + massive training on huge unlabeled data** (billions/trillions of tokens). That **whole pipeline** creates a **new base** model (Llama-scale, GPT-scale, etc.). Expensive; mostly big labs.

- **Continued pre-training** — You **start from an existing base checkpoint** (BERT, TinySolar, Llama, …) and train **further** on **domain** unlabeled text. You are **not** redoing foundation pre-training from random init at that scale — you are **nudging** an already-trained model.

**The part to remember:**

> **True pre-training** ≈ *new base* = **NN (specified architecture) + huge unlabeled data + massive compute**.  
> **Continued pre-training** ≈ *same lineage, new domain* = **existing pretrained weights + more unlabeled data in your niche** (your `ai-pretraining` / TinySolar story).

**Where `deep_neural_network.py` fits:** it teaches **how a network is constructed**. It is **not**, by itself, “true pre-training” — you’d still need the **massive data + training run** to match what Meta/xAI/Google call pre-training.

### 2. Where does `deep_neural_network.py` sit in the learning path?

**At the very beginning conceptually** — before you skip straight to Hugging Face checkpoints.

- It shows **how** a neural net is built (layers, tensors, training step).
- Repos like **`ai-pretraining`**, **`full-fine-tuning`**, **`peft-fine-tuning`** usually **load a ready-made pretrained model** (BERT, TinySolar, …) and **do not** redefine a full LM architecture from scratch in that repo.

So: **architecture literacy** (`deep_neural_network.py`) → later courses assume **someone else already ran** foundation pre-training (or you do **continued** pre-training / fine-tuning on top).

---

## Learning path (your repos)

Vertical companion to the [ASCII map](#conceptual-map-ascii) — **order you feel in the curriculum**, not chronological industry history:

```
  deep_neural_network.py          ← architecture + small-data training (education)
            │
            ▼
  Load a BASE checkpoint          ← BERT, Llama, TinySolar, … (foundation PT already done elsewhere)
            │
            ├─► Continued PT      →  ai-pretraining  (unlabeled domain)
            │
            └─► Task adaptation
                    ├─► Full FT   →  Week-3/full-fine-tuning
                    └─► PEFT      →  Week-3/peft-fine-tuning
```

---
