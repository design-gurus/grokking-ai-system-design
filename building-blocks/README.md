# Building blocks of AI systems

The reusable parts. Learn each one once, then assemble answers to questions you have never seen. Each file is short: what it is, when to use it, the trade-offs, and how to talk about it in an interview.

## Data and training

| Building block | What it solves | Status |
|----------------|----------------|--------|
| [Feature stores](feature-stores.md) | Serving the same feature values in training and in production | Written |
| Where training data comes from | Labels, and the delay before they arrive | Planned |
| Embeddings | Turning items and text into vectors you can compare | Planned |
| Two-tower retrieval | Narrowing millions of candidates to hundreds, fast | Planned |
| Multi-stage ranking | Retrieval, then ranking, then business rules | Planned |
| Model registry and rollout | Shipping a model without shipping a regression | Planned |
| Drift detection and retraining | Catching a model that got worse on its own | Planned |
| Online experiments | Deciding a launch with an A/B test, not an offline score | Planned |

## Retrieval

| Building block | What it solves | Status |
|----------------|----------------|--------|
| [Vector indexes](vector-indexes.md) | Finding similar items among billions, in milliseconds | Written |
| Chunking strategies | Splitting documents so retrieval can find the right passage | Planned |
| Hybrid search | Combining keyword and vector search, because each misses different things | Planned |
| Reranking | Buying precision at the top of the list with a second model | Planned |
| Grounding and citations | Making an answer traceable to a source | Planned |

## Serving

| Building block | What it solves | Status |
|----------------|----------------|--------|
| [Continuous batching and the KV cache](continuous-batching-and-kv-cache.md) | Getting more tokens per second out of the same GPU | Written |
| Batch vs online inference | Choosing between cheap and fresh | Planned |
| Model serving and autoscaling | Handling a traffic spike without a GPU for every user | Planned |
| Speculative decoding | Generating several tokens per model step | Planned |
| Quantization and distillation | Making a model smaller with a known quality cost | Planned |
| Semantic caching | Reusing an answer for a question that is not identical | Planned |
| Model gateway and routing | One entry point for many models, with fallback | Planned |

## LLM products and agents

| Building block | What it solves | Status |
|----------------|----------------|--------|
| Prompt vs fine-tune vs RAG | Closing a capability gap the right way | See the [cheat sheet](../cheat-sheets/rag-vs-fine-tuning-vs-prompting.md) |
| Tool calling | Letting a model act, not just answer | Planned |
| The agent loop and its bounds | Stopping a loop that would otherwise run forever | Planned |
| Agent memory | Carrying context across steps without carrying everything | Planned |
| Multi-agent orchestration | When splitting into several agents helps, and when it does not | Planned |
| Guardrails and safety filters | Blocking bad input and bad output, with a measured cost | Planned |
| Human in the loop | Routing the uncertain cases to a person | Planned |

## Cost

| Building block | What it solves | Status |
|----------------|----------------|--------|
| Token cost modeling | Knowing the monthly bill before you build | See [numbers you should know](../cheat-sheets/ai-numbers.md) |
| GPU capacity planning | Sizing a fleet from a traffic estimate | Planned |

## Contributing one

Copy [_template.md](_template.md), fill in every section, and add a row to the table above. Keep it to one concept per file. See [CONTRIBUTING.md](../CONTRIBUTING.md).
