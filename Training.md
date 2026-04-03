Real-World Examples (2026)

- Pre-training: xAI building Grok-3 from scratch on massive data.
- Continued Pre-training: Upstage taking a small model and training it more on Korean + code → TinySolar.
- Full Fine-Tuning: You take TinySolar and train all 248M weights on 10k instruction examples → a chatbot that follows your style perfectly.
- PEFT: Same as above but using LoRA → trains in 30 minutes on a single A100 instead of days.

Bottom line for you right now:

- ai-pretraining repo → **Continued Pre-training** (self-supervised, updates all weights).  
- ai-transformer/week-3  → **full-fine-tuning** repo → Full Fine-Tuning (usually supervised, updates all weights).

Both are after the original pre-training has already happened.



```
Start
  ↓
[Pre-training] ──────→ Base Model (e.g. BERT, Llama-3, TinySolar)
                       │
                       ├── Continued Pre-training ──→ Domain-specialized model (your ai-pretraining repo). Make learn Korean, Python
                       │
                       ├── Full Fine-Tuning ────────→ Task-specific model (your Week-3 full-fine-tuning repo). All weights updated. Costly
                       │
                       └── PEFT (LoRA/QLoRA) ────────→ Task-specific model (cheapest). Only few wights impacted.
```

### Clarification

Nine-column Markdown tables often force **horizontal scrolling**. Below: a **narrow summary table** you can read in one glance, then **stacked detail blocks** (same information, full width).

#### At a glance

| Style | Builds NN from scratch? | Updates all weights? | Typical compute |
| ----- | ----------------------- | -------------------- | --------------- |
| Pre-training | Yes | Yes | Very high (1000s of GPUs) |
| Continued pre-training | No | Yes | Medium (few GPUs) |
| Full fine-tuning | No | Yes | High (multi-GPU) |
| PEFT (LoRA / QLoRA) | No | No (~0.1–5% params) | Low (1 GPU / Colab OK) |
| From-scratch NN | Yes | Yes | Low–medium |

#### Pre-training

- **What it is:** Massive first training phase to build a foundational/base model from (near-)random weights.
- **Data:** Huge unlabeled text (trillions of tokens).
- **Your repo:** None (labs like Meta, xAI, Google-scale).
- **Example models:** Llama 3, GPT-4–class, Gemini, Grok — large foundation LMs.
- **Typical use cases:** General chat, code, reasoning, multimodal bases; starting point for later full FT, PEFT, or distillation.

#### Continued pre-training

- **What it is:** Start from an existing base and train further on new domain-heavy **unlabeled** text.
- **Data:** More unlabeled text (e.g. Python, Korean, legal, biomedical corpora).
- **Your repo:** `ai-pretraining` (TinySolar-248M + Python scripts).
- **Example models:** TinySolar-248M (Korean + code), CodeLlama-style extensions of Llama, other “second stage” checkpoints.
- **Typical use cases:** Shift vocabulary and style toward finance, law, medicine, multilingual, or code **before** task-specific fine-tuning.

#### Full fine-tuning (full training)

- **What it is:** Pre-trained model + **every weight** updated on task-specific (often labeled) data.
- **Data:** Smaller labeled sets — classification, QA, instructions, etc.
- **Your repo:** `ai-transformers/Week-3/full-fine-tuning` (BERT on IMDb & SQuAD).
- **Example models:** BERT-base in your labs; Llama / Mistral fully updated on instruction or domain JSONL.
- **Typical use cases:** Sentiment, extractive QA, NER, custom instruction chatbots when you can afford full-weight training.

#### PEFT (LoRA, QLoRA, …)

- **What it is:** Pre-trained model + **small adapter** weights; base mostly frozen.
- **Data:** Same kinds as full fine-tuning (labeled / instruction data).
- **Your repo:** `ai-transformers/Week-3/peft-fine-tuning` (LoRA / QLoRA examples).
- **Example models:** Llama 3 8B, Mistral 7B, Qwen2.5 — adapters on top of the same families you’d full FT.
- **Typical use cases:** Fast iteration, low VRAM, many small vertical adapters (support triage, tone, tagging) from one frozen base.

#### From-scratch neural network

- **What it is:** You define the full architecture (layers, shapes) yourself — educational or small-scale training.
- **Data:** Any; usually a **small** dataset for learning or demos.
- **Your repo:** `ai-deep-learning/src/core/deep_neural_network.py`.
- **Example models:** Custom MLPs; small CNNs on MNIST-style data in tutorials.
- **Typical use cases:** Teaching backprop and layer design; baselines and ablations; tiny embedded or specialized models where a pretrained LM is not the goal.


1. Is pretraining same as building own NN model from scratch?

No, they are not the same, but they are related.

Building own NN from scratch = You manually define the architecture (layers, attention heads, embeddings, etc.) using torch.nn.Module. This is what your deep_neural_network.py does.
Pre-training = After you have the architecture, you train it on massive unlabeled data with random initialization. This is the extremely expensive first step that only big companies do.

In practice:

True Pre-training = Building NN from scratch + training on trillions of tokens.
Your deep_neural_network.py = Only the “building architecture” part (for learning/education). It usually doesn’t do full-scale pre-training.

1. Where does deep_neural_network.py fit?

It sits at the very beginning — it teaches you how the Neural Network is actually constructed. All other repos (ai-pretraining, full-fine-tuning, peft-fine-tuning) skip this step and directly load a ready-made pre-trained model (BERT, TinySolar, etc.).

### Simple Flow – Complete Picture

```
Build Neural Network Architecture
        ↓
   (deep_neural_network.py — from scratch)

        ↓
Initialize Weights
   ├── Random → True Pre-training (big labs only)
   └── Pre-trained weights → Continued Pre-training / Fine-Tuning / PEFT

        ↓
Training Phase
   ├── Continued Pre-training   → ai-pretraining repo
   ├── Full Fine-Tuning         → Week-3/full-fine-tuning
   └── PEFT (LoRA)              → Week-3/peft-fine-tuning
```

