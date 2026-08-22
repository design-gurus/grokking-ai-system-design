# Deep dives

Real systems and landmark papers, summarized at interview altitude. Enough to name the idea, explain the trade-off, and answer a follow-up. Not enough to reimplement them, which is not what the round is for.

## Serving and models

| System | The idea worth knowing | Status |
|--------|------------------------|--------|
| [PagedAttention and vLLM](paged-attention-vllm.md) | LLM serving throughput is a memory management problem | Written |
| FlashAttention | Attention is limited by memory reads, not arithmetic | Planned |
| Mixture of experts | Grow parameters without growing the cost of one token | Planned |
| LoRA | Fine-tune a small add-on instead of the whole model | Planned |
| RLHF | Turning human preference into a training signal | Planned |
| Scaling laws | Why model size, data, and compute are chosen together | Planned |
| ZeRO and tensor parallelism | Training a model that does not fit on one GPU | Planned |

## Retrieval

| System | The idea worth knowing | Status |
|--------|------------------------|--------|
| HNSW | A navigable graph that turns search into a walk | Planned |
| DiskANN | Billion-scale search when the index cannot fit in memory | Planned |
| ColBERT | Late interaction: the accuracy of a cross-encoder at closer to vector-search speed | Planned |
| BM25 and hybrid scoring | Why keyword search still beats embeddings on exact terms | Planned |

## Production ML platforms

| System | The idea worth knowing | Status |
|--------|------------------------|--------|
| [Uber Michelangelo](uber-michelangelo.md) | The reference shape of an internal ML platform | Written |
| Meta DLRM | Recommendation at a scale where embedding tables dominate | Planned |
| YouTube two-tower retrieval | Splitting recommendation into retrieval then ranking | Planned |
| ByteDance Monolith | Online training with collisionless embedding tables | Planned |
| Ray | One framework for distributed training, tuning, and serving | Planned |
| Feast | The open-source feature store shape | Planned |

## Adding one

Copy [_template.md](_template.md). Keep it short: what it is, the problem, the key ideas, the trade-off, and what to say in an interview.
