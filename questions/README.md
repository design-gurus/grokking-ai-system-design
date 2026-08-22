# AI design question catalog

Every walkthrough uses the same [seven steps](../cheat-sheets/ai-system-design-framework.md) and stays at the level of approach, key decisions, and trade-offs. For full worked solutions and timed practice, see the [course](https://www.designgurus.io/course/grokking-the-ai-system-design-interview).

Questions are grouped by family, because the family tells you which step takes the most time.

## Classic ML

The system predicts something. The data step is the long one.

| Question | Difficulty | Key building blocks | Status |
|----------|-----------|---------------------|--------|
| [Design feed ranking](design-feed-ranking.md) | Medium | Multi-stage ranking, feature stores, online experiments | Written |
| [Design a recommendation system](design-a-recommendation-system.md) | Medium | Candidate generation, two-tower retrieval | Written |
| [Design search ranking](design-search-ranking.md) | Medium | Hybrid retrieval, reranking, click models | Written |
| [Design ad click prediction](design-ad-click-prediction.md) | Hard | Calibration, real-time features, auctions | Written |
| [Design fraud detection](design-fraud-detection.md) | Hard | Delayed labels, class imbalance, human review | Written |
| [Design harmful content detection](design-harmful-content-detection.md) | Hard | Risk funnel, action ladder, adversarial drift | Written |
| [Design people you may know](design-people-you-may-know.md) | Hard | Graph features, candidate generation at scale | Written |
| [Design ETA prediction](design-eta-prediction.md) | Medium | Streaming features, asymmetric loss | Written |
| [Design visual search](design-visual-search.md) | Medium | Image embeddings, vector indexes | Written |

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

Platform questions use the same seven steps, with step 4 as the core design choice rather than a model choice.

| Question | Difficulty | Key building blocks | Status |
|----------|-----------|---------------------|--------|
| [Design an LLM inference platform](design-an-llm-inference-platform.md) | Hard | Continuous batching, autoscaling, multi-tenancy | Written |
| [Design an LLM gateway](design-an-llm-gateway.md) | Medium | Routing, fallback, quotas, cost attribution | Written |
| [Design a GPU cluster scheduler](design-a-gpu-cluster-scheduler.md) | Hard | Gang scheduling, preemption, fairness | Written |
| [Design a feature store](design-a-feature-store.md) | Medium | Offline and online stores, point-in-time joins | Written |
| [Design a model evaluation pipeline](design-a-model-evaluation-pipeline.md) | Medium | Offline sets, judges, regression gates | Written |
| [Design a vector database](design-a-vector-database.md) | Hard | ANN indexes, sharding, filtered search | Written |
| [Design a prompt management service](design-a-prompt-management-service.md) | Medium | Versioning, experiments, rollback | Written |
| [Design a data labeling platform](design-a-data-labeling-platform.md) | Medium | Quality control, agreement, active learning | Written |
| [Design model monitoring](design-model-monitoring.md) | Medium | Drift, delayed labels, alerting | Written |

## Adding a question

Copy [_template.md](_template.md) and add a row to the right table. Keep the solution at approach level. Full paid-grade solutions belong in the course, not here. See [CONTRIBUTING.md](../CONTRIBUTING.md).
