# Grokking AI System Design

> The free, open companion to **Grokking the AI System Design Interview** by [DesignGurus.io](https://www.designgurus.io/course/grokking-the-ai-system-design-interview), created by Arslan Ahmad.

A classic system design interview asks how data moves. An AI system design interview asks how a system that learns is built, served, measured, and paid for. This repository is a free index of the building blocks, the questions, and the reference sheets for that round. For worked solutions, diagrams, and timed practice, see the full course.

[![GitHub stars](https://img.shields.io/github/stars/design-gurus/grokking-ai-system-design?style=social)](https://github.com/design-gurus/grokking-ai-system-design/stargazers)
[![Last commit](https://img.shields.io/github/last-commit/design-gurus/grokking-ai-system-design)](https://github.com/design-gurus/grokking-ai-system-design/commits/main)
[![License: CC BY 4.0](https://img.shields.io/badge/License-CC%20BY%204.0-lightgrey.svg)](LICENSE)
[![PRs welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](CONTRIBUTING.md)

---

## Contents

- [What is AI system design?](#what-is-ai-system-design)
- [The three question families](#the-three-question-families)
- [How an AI round differs](#how-an-ai-round-differs-from-a-normal-system-design-round)
- [FAQ](#faq)
- [The seven-step framework](#the-seven-step-framework)
- [Building blocks](#building-blocks)
- [Design questions](#design-questions)
- [Cheat sheets](#cheat-sheets)
- [Evaluation](#evaluation)
- [Deep dives](#deep-dives)
- [Study roadmaps](#study-roadmaps)
- [Glossary](#glossary)
- [What is coming next](#what-is-coming-next)
- [Contributing](#contributing)

---

## What is AI system design?

AI system design is the interview round where the system you design contains a model. The model can be a ranking model, a large language model, or a chain of tool calls. The round still covers scale, storage, and latency. It adds four questions that a classic design never faces.

1. Where does the training data come from?
2. How do you know the output is any good?
3. What happens when the model gets worse over time?
4. What does one request cost?

Candidates who answer only the first three steps of a classic framework fail this round. They describe boxes and arrows, then stop before data, evaluation, and cost. Those last three are where the signal is.

## The three question families

Almost every question in this round belongs to one of three families. Each family has its own shape, its own metrics, and its own failure modes.

| Family | The question sounds like | What it really tests |
|--------|--------------------------|----------------------|
| Classic ML | "Design a feed ranking system" | Features, labels, training and serving skew, offline and online metrics |
| GenAI and LLM products | "Design a ChatGPT-style assistant" | Prompting or retrieval or fine-tuning, token cost, latency, safety |
| Agentic systems | "Design a customer support agent" | Tool calls, bounds on the loop, memory, evaluation of a whole trajectory |

The families share one skeleton, the [seven-step framework](#the-seven-step-framework). They differ in which step takes the most time. Classic ML spends it on data. LLM products spend it on the serving path and the cost. Agents spend it on control and evaluation.

## How an AI round differs from a normal system design round

| Classic system design | AI system design |
|-----------------------|------------------|
| Correctness is defined by a spec | Quality is a number you have to choose and defend |
| The system is right or it has a bug | The system is right 92 percent of the time, and you argue about the other 8 |
| Behavior changes when you deploy code | Behavior changes when data changes, with no deploy at all |
| Cost is servers and storage | Cost is GPUs and tokens, and it scales with every single request |
| Testing means unit and integration tests | Testing means an offline set, an online experiment, and a guardrail metric |
| The data model is tables | The data model is tables, features, embeddings, and prompts |

## FAQ

**Is there an AI system design PDF or book?**

There is no official PDF or printed book of Grokking the AI System Design Interview. Copies that circulate online are unofficial and out of date. This repository is the free way to read the material, in your browser, with no account. The full course is at [Grokking the AI System Design Interview](https://www.designgurus.io/course/grokking-the-ai-system-design-interview).

**Do I need a machine learning background?**

No. This round grades system judgment, not math. You are never asked to derive gradient descent. You are asked where the labels come from, what you measure, and what it costs. A backend engineer who can answer those three questions beats a researcher who cannot. If you are converting from backend work, start with the [backend engineer roadmap](roadmaps/backend-engineer-to-ai-system-design.md).

**Is this ML system design or GenAI system design?**

Both. Companies use the terms loosely and mix the families in one loop. A recommendation question and a RAG question can appear in the same day, from the same interviewer. This repository covers all three families.

**How is this different from the [Grokking System Design](https://github.com/design-gurus/grokking-system-design) repo?**

That repo is the pattern-based guide to classic distributed systems interviews. Start there if your round has no model in it. A few questions appear in both repos. The version there is written for a general round and leads with architecture. The version here is written for an AI round and leads with data, evaluation, and cost.

## The seven-step framework

One structure covers all three families. Full breakdown with timings: [cheat-sheets/ai-system-design-framework.md](cheat-sheets/ai-system-design-framework.md).

1. **Frame the problem.** Turn a vague product ask into one prediction or one generation task.
2. **Define success.** Name the offline metric, the online metric, and the guardrail metric.
3. **Map the data.** Say where labels come from and how features reach the serving path.
4. **Pick the model.** Start with the simplest thing that works, then name the upgrade.
5. **The serving path.** Walk one request end to end, inside a stated latency budget.
6. **Keep it working.** Monitoring, drift, retraining, rollback, and the feedback loop.
7. **Count the costs.** Tokens, GPUs, and storage, per request and per month.

## Building blocks

The reusable parts of an AI system. Learn these once and you can assemble an answer to most questions.

| Building block | What it solves | Status |
|----------------|----------------|--------|
| [Feature stores](building-blocks/feature-stores.md) | Serving the same feature values in training and in production | Written |
| [Vector indexes](building-blocks/vector-indexes.md) | Finding similar items among billions, in milliseconds | Written |
| [Continuous batching and the KV cache](building-blocks/continuous-batching-and-kv-cache.md) | Getting more tokens per second out of the same GPU | Written |

More are on the way. See [what is coming next](#what-is-coming-next). To add one, copy [building-blocks/_template.md](building-blocks/_template.md).

## Design questions

Each walkthrough stays at the level of approach, key decisions, and trade-offs, using the seven steps. Full catalog in [questions/](questions/).

- [Design feed ranking](questions/design-feed-ranking.md), the classic ML family.
- [Design enterprise document Q and A](questions/design-enterprise-document-qa.md), the retrieval family.
- [Design a customer support agent](questions/design-a-customer-support-agent.md), the agentic family.

## Cheat sheets

- [The AI system design framework](cheat-sheets/ai-system-design-framework.md): the seven steps, with timings and the sentence to say at each one.
- [RAG vs fine-tuning vs prompting](cheat-sheets/rag-vs-fine-tuning-vs-prompting.md): the most common follow-up in the round, decided in one table.
- [Numbers you should know](cheat-sheets/ai-numbers.md): token costs, GPU memory math, and the latency budget of a generated answer.

## Evaluation

Evaluation is the step candidates skip and interviewers grade hardest. See [evals/](evals/), starting with [LLM as judge](evals/llm-as-judge.md).

## Deep dives

The systems and papers worth being able to name. See [deep-dives/](deep-dives/).

- [PagedAttention and vLLM](deep-dives/paged-attention-vllm.md): why serving throughput was a memory problem.
- [Uber Michelangelo](deep-dives/uber-michelangelo.md): the reference shape of an internal ML platform.

## Study roadmaps

- [Backend engineer to AI system design](roadmaps/backend-engineer-to-ai-system-design.md): for engineers who have shipped services but never a model.
- More plans in [roadmaps/](roadmaps/).

## Glossary

New to the vocabulary? Start with the [glossary](glossary.md). Reading list: [resources.md](resources.md).

## What is coming next

This repository is being built in batches. Planned, in order:

1. The rest of the building blocks. Embeddings, two-tower retrieval, multi-stage ranking, drift and retraining, and online experiments. Then speculative decoding, quantization, semantic caching, model gateways, tool calling, agent memory, guardrails, and GPU capacity planning.
2. The full question catalog across all three families, plus a practice bank.
3. Comparison sheets: vector databases, GPUs, serving stacks, embedding models, open vs closed models.
4. Deep dives on retrieval, training, and production ML platforms.
5. Evaluation guides per system type.
6. Company pages for teams that run an AI round.

Want to help? Pick one from the list and open a pull request. See [CONTRIBUTING.md](CONTRIBUTING.md).

## Go deeper: the full course

This repository is a free summary. The course has the worked solutions, the diagrams, the quizzes, and eight timed scenario walkthroughs.

- Course: [Grokking the AI System Design Interview](https://www.designgurus.io/course/grokking-the-ai-system-design-interview)
- AI foundations: [Grokking Modern AI Fundamentals](https://www.designgurus.io/course/grokking-modern-ai-fundamentals)
- Classic system design: [Grokking the System Design Interview](https://www.designgurus.io/course/grokking-the-system-design-interview)
- Practice live: [Mock interviews with ex-FAANG engineers](https://www.designgurus.io/mock-interviews)

## Newsletter

System design and interview tips, straight to your inbox. [Subscribe on Substack](https://designgurus.substack.com/)

## Contributing

Contributions are welcome. See [CONTRIBUTING.md](CONTRIBUTING.md). If this repository helps you, please star it so more engineers can find it.

## License

Content is licensed under [Creative Commons Attribution 4.0 (CC BY 4.0)](LICENSE). You may share and adapt with attribution.

## About

Maintained by [DesignGurus.io](https://www.designgurus.io/), the home of the original Grokking courses by Arslan Ahmad.
