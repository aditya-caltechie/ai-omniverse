# Fine-Tuning Large Language Models

Fine-tuning lets you take a powerful pre-trained model and adapt it to your own data or tasks. The right technique depends heavily on your **hardware budget**, **data size**, and **how much you want to change the model’s behavior**.

---

## Key Terms at a Glance

| **Term**           | **Category**        | **What it describes**                                                        |
|--------------------|---------------------|------------------------------------------------------------------------------|
| **SFT**            | Learning Paradigm   | Using labeled *input/output* data to teach a specific behavior.             |
| **Full Fine-Tuning** | Resource Method   | Modifying **100%** of the model's weights during SFT.                       |
| **LoRA / QLoRA**   | Resource Method     | Modifying **< 1%** of the model's weights during SFT (parameter-efficient). |

### Stages vs. What They Learn

| **Stage**    | **What is learned?**              | **Analogy**                                  |
|-------------|------------------------------------|----------------------------------------------|
| **SFT**     | Domain knowledge & format         | Learning the textbook for a class.           |
| **DPO / RLHF** | Style, safety, & preference    | Taking practice exams and getting a grade.   |

---

## 1. Full Fine-Tuning (Instruction Tuning)

**Idea:** Update **all** of the model’s parameters on your custom dataset.

- **How it works:**
  - Start from a pre-trained base model.
  - Continue training (often with supervised learning) on your task-specific or instruction-style data.
- **Pros:**
  - Can yield the **best task-specific performance**.
  - Model fully internalizes your domain/task.
- **Cons:**
  - **Very resource-intensive** (requires lots of VRAM, time, and storage).
  - Produces a **full-size new model** (same order of GB as the base model).
  - Higher risk of **catastrophic forgetting** (losing general knowledge) if the dataset is narrow.

---

## 2. Parameter-Efficient Fine-Tuning (PEFT)

PEFT methods aim to get close to full fine-tuning performance while only training a **tiny fraction** of the parameters.

### 2.1 LoRA (Low-Rank Adaptation)

**Idea:** Insert small trainable matrices into existing layers instead of updating the huge original weight matrices.

- **How it works:**
  - For selected layers (e.g., attention or MLPs), decompose weight updates into **low-rank matrices** \(A, B\).
  - During training, **only A and B are updated**; the base weights stay frozen.
- **Key benefits:**
  - Updates **< 1%** of parameters.
  - **VRAM usage drops dramatically** (often ~10× less than full fine-tuning).
  - The trained LoRA weights are **small adapter files** that can be swapped on top of the same base model.

### 2.2 QLoRA (Quantized LoRA)

**Idea:** Combine quantization + LoRA to make fine-tuning possible on a **single consumer GPU**.

- **How it works:**
  - Load the base model in **4-bit quantized** form (significantly reducing memory).
  - Apply LoRA adapters on top and train only those adapters.
- **Key benefits:**
  - Enables fine-tuning of multi‑billion parameter models on **laptop / single-GPU setups**.
  - Keeps a **good tradeoff** between quality and memory usage.

---

## 3. Prompt-Based Fine-Tuning

These approaches **don’t modify the model weights directly**. Instead, they learn how to **steer** the model via trainable prompts.

### 3.1 Prompt Tuning

- Learn a trainable **“soft prompt”** (a set of continuous embeddings) that is prepended to the input.
- During training, **only the prompt embeddings are updated**, not the model weights.
- Extremely parameter-efficient (often **~0.01%** of parameters).

### 3.2 Prefix Tuning

- Extends the idea of prompt tuning by attaching trainable vectors as **prefixes to each layer** of the transformer.
- Gives the model more “levers” at multiple depths, while still keeping the number of trainable parameters small.

**When to use prompt / prefix tuning:**

- You have **very limited compute**.
- You want to support **many small specializations** on top of a shared base model.

---

## 4. Preference Alignment (RLHF & DPO)

These methods align a model with **human preferences** (helpfulness, safety, style) rather than with a single supervised label.

### 4.1 RLHF (Reinforcement Learning from Human Feedback)

Standard post-training pipeline used in many chat assistants.

1. **Supervised Fine-Tuning (SFT):** Train the base model on high-quality instruction–response examples.
2. **Reward Model:** Train a separate model to score responses based on human preference rankings.
3. **RL Optimization (e.g., PPO):** Use the reward model to **reinforce** outputs that humans prefer and penalize bad ones.

- **Pros:** Powerful and expressive; can shape nuanced behaviors.
- **Cons:**
  - Quite **complex** (two models, RL loop, stability issues).
  - Requires **lots of human-labelled preference data**.

### 4.2 DPO (Direct Preference Optimization)

A simpler alternative to RLHF that skips the explicit reward model and RL phase.

- **Idea:**
  - Treat preference learning as a **direct optimization problem** over pairs of responses.
  - Given a **preferred** answer and a **rejected** answer, optimize the model to increase the likelihood of the preferred one vs. the rejected one.
- **Benefits:**
  - **No separate reward model** and **no RL loop** required.
  - Easier to implement and often more stable in practice.

---

## 5. Comparison Summary

| Technique                   | Parameters Updated | Hardware Demand       | Conceptual / Operational Complexity |
|----------------------------|--------------------|-----------------------|--------------------------------------|
| **Full Fine-Tuning**       | ~100%              | Very High             | Medium                               |
| **LoRA / QLoRA (PEFT)**    | < 1%               | Low → Medium          | Medium                               |
| **Prompt / Prefix Tuning** | ~0.01%             | Very Low              | High (prompt design & interpretation) |
| **DPO / RLHF**             | Varies             | High                  | Very High                            |

In practice, most modern LLM applications follow a **two-stage pattern**:

1. **Stage 1 – SFT for task + domain**: Use **SFT with PEFT (LoRA / QLoRA)** on your data to efficiently teach the model domain knowledge and output format (avoid full fine-tuning unless you truly need it and have the hardware).
2. **Stage 2 – Alignment for polish**: Optionally apply **DPO / RLHF** on preference data to polish **style, safety, and helpfulness** on top of the SFT model.

You can often ship a strong product after Stage 1 alone; Stage 2 is most valuable when user experience and safety constraints are critical.

