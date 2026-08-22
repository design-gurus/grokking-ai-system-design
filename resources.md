# Resources

Free reading that pairs well with this repository. Papers are listed for the idea they contributed, not because you need to read them end to end.

## Start here

- [Grokking the AI System Design Interview](https://www.designgurus.io/course/grokking-the-ai-system-design-interview), the course this repository accompanies.
- [Grokking Modern AI Fundamentals](https://www.designgurus.io/course/grokking-modern-ai-fundamentals), if the model vocabulary itself is new.
- [Grokking the System Design Interview](https://www.designgurus.io/course/grokking-the-system-design-interview), for the classic distributed systems half that every AI round still assumes.
- [Grokking System Design](https://github.com/design-gurus/grokking-system-design), the free companion repository for classic system design.

## Papers worth knowing by name

Serving:

- Efficient Memory Management for Large Language Model Serving with PagedAttention, the vLLM paper. See the [deep dive](deep-dives/paged-attention-vllm.md).
- FlashAttention, on making attention read memory less.
- Orca, on continuous batching at the level of a single model step.

Retrieval:

- Efficient and robust approximate nearest neighbor search using HNSW graphs.
- DiskANN, for billion-scale search when the index does not fit in memory.
- ColBERT, on late interaction between query and document.
- Dense Passage Retrieval, the paper that made two-tower retrieval standard.

Recommendation:

- Deep Neural Networks for YouTube Recommendations, the reference for splitting retrieval from ranking.
- Wide and Deep Learning for Recommender Systems.
- Deep Learning Recommendation Model, on the scale of embedding tables in production.

Language models and training:

- Attention Is All You Need.
- Training Language Models to Follow Instructions with Human Feedback, the InstructGPT paper.
- LoRA, on low-rank adaptation.
- Training Compute-Optimal Large Language Models, the Chinchilla paper.
- ZeRO, on the memory arithmetic of training large models.

Agents:

- ReAct, on interleaving reasoning and tool use.
- Toolformer, on models learning to call APIs.

## Engineering blogs

These are where production detail lives, and interviewers often work at these companies.

- Uber Engineering, for Michelangelo and the feature store shape.
- Meta Engineering, for ranking and recommendation at scale.
- Netflix Tech Blog, for recommendation and experimentation.
- Pinterest Engineering, for visual search and retrieval.
- Spotify Engineering, for music recommendation.
- LinkedIn Engineering, for people recommendation and graph features.
- DoorDash Engineering, for ETA prediction and real-time features.

## Tools worth being able to name

You are not asked to configure these, only to know what category each one fills.

| Category | Examples |
|----------|----------|
| Serving runtimes | vLLM, TGI, TensorRT-LLM, Triton |
| Vector search | FAISS, HNSWlib, pgvector, Milvus, Weaviate, Pinecone |
| Feature stores | Feast, Tecton, and internal platforms like Michelangelo |
| Orchestration | Ray, Kubeflow, Airflow, Temporal |
| Experiment and model tracking | MLflow, Weights and Biases |
| Evaluation | Ragas, promptfoo, and internal judge harnesses |

## DesignGurus reading

- [DesignGurus blog](https://www.designgurus.io/blog)
- [Mock interviews with ex-FAANG engineers](https://www.designgurus.io/mock-interviews)
- [Newsletter on Substack](https://designgurus.substack.com/)
