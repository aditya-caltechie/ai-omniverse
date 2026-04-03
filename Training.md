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
[Pre-training] ──────→ Base Model (e.g. Llama-3, TinySolar)
                       │
                       ├── Continued Pre-training ──→ Domain-specialized model (your ai-pretraining repo). Make learn Korean, Python
                       │
                       ├── Full Fine-Tuning ────────→ Task-specific model (your Week-3 full-fine-tuning repo). All weights updated. Costly
                       │
                       └── PEFT (LoRA/QLoRA) ────────→ Task-specific model (cheapest). Only few wights impacted.
```

### Clarification: 


| Term | What it means | Builds NN from scratch? | Updates all weights? | Data used | Typical compute | Your repo example | Example models | Typical use cases |
| ---- | ------------- | --------------------- | -------------------- | --------- | --------------- | ----------------- | -------------- | ----------------- |
| **Pre-training** | First massive training phase to create a foundational/base model from random weights | Yes | Yes | Huge unlabeled text (trillions of tokens) | Extremely high (1000s of GPUs) | None of your repos (only big labs like Meta, xAI, Google do this) | **Llama 3**, **GPT-4 class**, **Gemini**, **Grok** — billion-parameter foundation LMs | General chat, code, reasoning, and multimodal bases; starting point for later FT, PEFT, or distillation |
| **Continued Pre-training** | Take an existing base model + train more on new domain-specific data | No | Yes | More unlabeled text (domain data, e.g. Python code) | Medium (few GPUs) | `ai-pretraining` repo (TinySolar-248m + Python scripts) | **TinySolar-248M** (Korean + code), **CodeLlama** (code extension of Llama), domain “second stage” checkpoints | Shift vocabulary and style toward **finance, law, biomedicine, multilingual**, or **more code** before task-specific fine-tuning |
| **Full Fine-Tuning (Full Training)** | Take a pre-trained model + update every weight on task-specific data | No | Yes | Smaller labeled task data (classification, QA, etc.) | High (multiple GPUs) | `ai-transformers/Week-3/full-fine-tuning` (BERT on IMDb & SQuAD) | **BERT-base** (IMDb sentiment, SQuAD QA in your labs); **Llama / Mistral** fully updated on instruction or domain JSONL | **Classification**, **extractive QA**, **NER**, **custom instruction** chatbots where you can afford full-weight updates |
| **PEFT (LoRA, QLoRA, etc.)** | Take a pre-trained model + update only a tiny fraction of parameters using adapters | No | No (only ~0.1–5%) | Same as fine-tuning (labeled task data) | Very low (even on 1 GPU or Colab) | `ai-transformers/Week-3/peft-fine-tuning` (contains examples with LoRA/QLoRA) | Same families as full FT: **Llama 3 8B**, **Mistral 7B**, **Qwen2.5** — adapters sit on top | **Fast iteration**, **low VRAM**, many **small vertical adapters** (support triage, tone, tags) from one frozen base |
| **From-Scratch NN** | Define and build the entire neural network architecture yourself | Yes | Yes | Any (usually small dataset for learning) | Low to Medium | `ai-deep-learning/src/core/deep_neural_network.py` | **Custom MLP** / small nets in course code; toy **CNNs** on MNIST-style data in tutorials | **Teaching** backprop and layers; **baselines** and **ablations**; tiny **embedded** or **specialized** models where a pretrained LM is not the goal |


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

