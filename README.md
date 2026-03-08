# ai-omniverse

<p align="center">
  <img src="assets/ai-omniverse-banner.png" alt="AI/ML learning: traditional ML to deep learning, Gen AI, agentic AI, and MLOps" width="680"/>
</p>
<p align="center"><em>Studying AI & agents for the future</em></p>

**One stop for AI/ML learning** — AL/ML learning Gym including traditional ML, deep learning, fine-tuning, Gen AI, Agentic AI. Everything from experiments to production with evaluation and observability.

This repo is a curated learning path covering the full spectrum of modern AI and machine learning:

- **Traditional ML** — fundamentals: regression, classification, clustering, feature engineering, and model evaluation.
- **Deep Learning** — neural networks, CNNs, RNNs, transformers, and training at scale.
- **Gen AI & LLMs** — building applications with large language models: prompting, RAG, fine-tuning, and deployment.
- **Agentic AI** — autonomous agents, tool use, multi-step reasoning, and agent frameworks.
- **MLOps** — experiment tracking, model versioning, pipelines, serving, and monitoring in production.

Course repos and materials are organized by track below.

---

## 1. LLM Core Track

| **Topic** | **Repo** | **Description** |
|-----------|----------|-----------------|
| LangChain Basics | [ai-langchain-intro](https://github.com/aditya-caltechie/ai-langchain-intro) | Introduction to LangChain: chains, prompts, output parsers, and connecting to LLMs. Covers the core building blocks for LLM applications. |
| Deep Learning | [ai-deep-learning](https://github.com/aditya-caltechie/ai-deep-learning) | Deep learning fundamentals with PyTorch/TensorFlow: neural networks, CNNs, RNNs, and training pipelines. Foundation for understanding how modern LLMs are built. |
| Fine-tuning | [ai-fine-tuning](https://github.com/aditya-caltechie/ai-fine-tuning) | Fine-tuning large language models (e.g. LoRA, QLoRA, adapters) on custom data. Covers data prep, training, and evaluation for domain-specific models. |
| RLHF pipeline | Todo | Yet to explore. |
| RAG | [ai-rag](https://github.com/aditya-caltechie/ai-rag) | Retrieval-augmented generation: vector stores, embeddings, and chaining retrievers with LLMs for knowledge-grounded answers. |
| Agentic RAG| [ai-agentic-rag](https://github.com/aditya-caltechie/ai-agentic-rag) | Agentic RAG: agents that decide when and how to search and reason over retrieved context using LangChain/LangGraph-style patterns. |
| AI Agent using Tools/loop & Workflows | [ai-deals2buy](https://github.com/aditya-caltechie/ai-deals2buy) | Capstone: AI agent for deals/shopping with LLM calls, tools, and agent logic. Two approaches: (a) loops & tools via `autonomous_planning_agent.py`, (b) workflow via `planning_agent.py`. |

---

## 2. Agentic Core Track

| **Topic** | **Repo** | **Description** |
|-----------|----------|-----------------|
| OpenAI Agents SDK | — | *Links to be added.* |
| MCP (Model Context Protocol) | [ai-mcp-autonomous-traders](https://github.com/aditya-caltechie/ai-mcp-autonomous-traders) | Autonomous trading agents using MCP: agents that use tools and context via the Model Context Protocol for trading workflows. |
| CrewAI Framework | [ai-crew-engineering-team](https://github.com/aditya-caltechie/ai-crew-engineering-team) | Multi-agent "engineering team" with CrewAI: role-based agents collaborating on tasks. Demonstrates CrewAI's agent and task APIs. |
| | [ai-crew-financial-researcher](https://github.com/aditya-caltechie/ai-crew-financial-researcher) | Financial research agent built with CrewAI: research tasks, tools, and structured outputs for finance use cases. |
| | [ai-crew-stock-picker](https://github.com/aditya-caltechie/ai-crew-stock-picker) | Stock-picking agent with CrewAI: agents that analyze and recommend stocks using external data and tools. |
| LangGraph | — | *Links to be added.* |
| Autogen | — | *Links to be added.* |



---

## 3. MLOps Track

| **Topic** | **Repo** | **Description** |
|-----------|----------|-----------------|
| Deploy Gen AI & Agentic AI in Production | [MLOps-production](https://github.com/aditya-caltechie/ed-donner-MLOps-production) | Course repo: deploy Gen AI and Agentic AI at scale in 4 weeks. Covers production deployment, guides, and week-by-week materials (Udemy companion). |

## Study Notes

| **Repo** | **Description** |
|----------|-----------------|
| [ai-tutorial-notes](https://github.com/aditya-caltechie/ai-tutorial-notes) | Consolidated notes and references from Udemy AI/ML courses. Quick lookup for concepts, commands, and patterns used across the tracks. |


# Appendix :

Something yet to explore RLHF :
```
Supervised Fine-Tuning (SFT)
 - High-quality instruction-response pairs
 - Teaches model SQL generation patterns
 ↓
Reward Model Training
 - Preference learning (good vs bad SQL)
 - Learns to score SQL quality
 ↓
PPO Optimization (RLHF)
 - Uses reward model as critic
 - Optimizes for better SQL generation
 ↓
Final PPO Model
 - Optimized for your RAG use case
 - Better at generating clean, correct SQL
```
