# AI design question catalog

Every walkthrough uses the same [seven steps](../cheat-sheets/ai-system-design-framework.md) and stays at the level of approach, key decisions, and trade-offs. For full worked solutions and timed practice, see the [course](https://www.designgurus.io/course/grokking-the-ai-system-design-interview).

Questions are grouped by family, because the family tells you which step takes the most time.

## Classic ML

The system predicts something. The data step is the long one.

| Question | Difficulty | Key building blocks | Status |
|----------|-----------|---------------------|--------|
| [Design feed ranking](design-feed-ranking.md) | Medium | Multi-stage ranking, feature stores, online experiments | Written |
| Design a recommendation system | Medium | Candidate generation, two-tower retrieval | Planned |
| Design search ranking | Medium | Hybrid retrieval, reranking, click models | Planned |
| Design ad click prediction | Hard | Calibration, real-time features, auctions | Planned |
| Design fraud detection | Hard | Delayed labels, class imbalance, human review | Planned |
| Design harmful content detection | Hard | Risk funnel, action ladder, adversarial drift | Planned |
| Design people you may know | Hard | Graph features, candidate generation at scale | Planned |
| Design ETA prediction | Medium | Streaming features, geospatial data | Planned |
| Design visual search | Medium | Image embeddings, vector indexes | Planned |

## GenAI and LLM products

The system generates something. The serving and cost steps are the long ones.

| Question | Difficulty | Key building blocks | Status |
|----------|-----------|---------------------|--------|
| [Design enterprise document Q and A](design-enterprise-document-qa.md) | Medium | RAG, chunking, permission filtering, grounding | Written |
| Design a ChatGPT-style assistant | Medium | Serving, streaming, conversation state, safety | Planned |
| Design a code assistant | Hard | Context assembly, latency budget, acceptance metrics | Planned |
| Design an AI search engine | Hard | Hybrid retrieval, citation, freshness | Planned |
| Design a text-to-image service | Medium | GPU fleets, queueing, safety filters | Planned |
| Design a meeting assistant | Medium | Speech, chunking, summarization evaluation | Planned |
| Design an AI tutor | Medium | Personalization, guardrails, cost per learner | Planned |

## Agentic systems

The system acts. The control and evaluation steps are the long ones.

| Question | Difficulty | Key building blocks | Status |
|----------|-----------|---------------------|--------|
| [Design a customer support agent](design-a-customer-support-agent.md) | Hard | Tool calling, bounds, escalation, trajectory evaluation | Written |
| Design a coding agent | Hard | Sandboxing, verification, long-running tasks | Planned |
| Design a research assistant | Hard | Planning, memory, source tracking | Planned |

## Platform and infrastructure

The system serves other teams' models. These come up in infrastructure loops.

| Question | Difficulty | Key building blocks | Status |
|----------|-----------|---------------------|--------|
| Design an LLM inference platform | Hard | Continuous batching, autoscaling, multi-tenancy | Planned |
| Design an LLM gateway | Medium | Routing, fallback, quotas, cost attribution | Planned |
| Design a GPU cluster scheduler | Hard | Gang scheduling, preemption, fairness | Planned |
| Design a feature store | Medium | Offline and online stores, point-in-time joins | Planned |
| Design a model evaluation pipeline | Medium | Offline sets, judges, regression gates | Planned |
| Design a vector database | Hard | ANN indexes, sharding, filtered search | Planned |
| Design a prompt management service | Medium | Versioning, experiments, rollback | Planned |
| Design a data labeling platform | Medium | Quality control, agreement, active learning | Planned |
| Design model monitoring | Medium | Drift, delayed labels, alerting | Planned |

## Adding a question

Copy [_template.md](_template.md) and add a row to the right table. Keep the solution at approach level. Full paid-grade solutions belong in the course, not here. See [CONTRIBUTING.md](../CONTRIBUTING.md).
