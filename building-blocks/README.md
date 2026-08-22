# Building blocks of AI systems

The reusable parts. Learn each one once, then assemble answers to questions you have never seen. Each file is short: what it is, when to use it, the trade-offs, and how to talk about it in an interview.

Twenty-seven blocks, grouped by where they sit in a system.

## Data and training

| Building block | What it solves |
|----------------|----------------|
| [Where training data comes from](where-training-data-comes-from.md) | Labels, what they cost, and how long they take to arrive |
| [Feature stores](feature-stores.md) | Serving the same feature values in training and in production |
| [Embeddings](embeddings.md) | Turning items and text into vectors you can compare |
| [Two-tower retrieval](two-tower-retrieval.md) | Narrowing millions of candidates to hundreds, fast |
| [Multi-stage ranking](multi-stage-ranking.md) | Retrieval, then ranking, then business rules |
| [Model registry and rollout](model-registry-and-rollout.md) | Shipping a model without shipping a regression |
| [Drift detection and retraining](drift-detection-and-retraining.md) | Catching a model that got worse on its own |
| [Online experiments](online-experiments.md) | Deciding a launch with an A/B test, not an offline score |

## Retrieval

| Building block | What it solves |
|----------------|----------------|
| [Vector indexes](vector-indexes.md) | Finding similar items among billions, in milliseconds |
| [Chunking strategies](chunking-strategies.md) | Splitting documents so retrieval can find the right passage |
| [Hybrid search](hybrid-search.md) | Combining keyword and vector search, because each misses different things |
| [Reranking](reranking.md) | Buying precision at the top of the list with a second model |
| [Grounding and citations](grounding-and-citations.md) | Making an answer traceable to a source |

## Serving

| Building block | What it solves |
|----------------|----------------|
| [Batch vs online inference](batch-vs-online-inference.md) | Choosing between cheap and fresh |
| [Model serving and autoscaling](model-serving-and-autoscaling.md) | Handling a traffic spike when replicas take minutes to start |
| [Continuous batching and the KV cache](continuous-batching-and-kv-cache.md) | Getting more tokens per second out of the same GPU |
| [Speculative decoding](speculative-decoding.md) | Generating several tokens per model step |
| [Quantization and distillation](quantization-and-distillation.md) | Making a model smaller with a known quality cost |
| [Semantic caching](semantic-caching.md) | Reusing an answer for a question that is not identical |
| [Model gateway and routing](model-gateway-and-routing.md) | One entry point for many models, with fallback and cost attribution |

## LLM products and agents

| Building block | What it solves |
|----------------|----------------|
| [Tool calling](tool-calling.md) | Letting a model act, not just answer |
| [The agent loop and its bounds](agent-loop-and-bounds.md) | Stopping a loop that would otherwise run forever |
| [Agent memory](agent-memory.md) | Carrying context across steps without carrying everything |
| [Multi-agent orchestration](multi-agent-orchestration.md) | When splitting into several agents helps, and when it does not |
| [Guardrails and safety filters](guardrails-and-safety-filters.md) | Blocking bad input and bad output, with a measured cost |
| [Human in the loop](human-in-the-loop.md) | Routing the uncertain cases to a person |
| Prompt vs fine-tune vs RAG | Closing a capability gap the right way. See the [cheat sheet](../cheat-sheets/rag-vs-fine-tuning-vs-prompting.md) |

## Cost

| Building block | What it solves |
|----------------|----------------|
| [GPU capacity planning](gpu-capacity-planning.md) | Sizing a fleet from a traffic estimate |
| Token cost modeling | Knowing the monthly bill before you build. See [numbers you should know](../cheat-sheets/ai-numbers.md) |

## Contributing one

Copy [_template.md](_template.md), fill in every section, and add a row to the right table above. Keep it to one concept per file. See [CONTRIBUTING.md](../CONTRIBUTING.md).
