# Reranking

> A second, slower model that reorders a short candidate list, and usually the cheapest quality win in a retrieval system.

## What it is

Retrieval returns 50 to 100 candidates in the right neighborhood. A reranker scores each one against the query with a much more accurate model, then keeps the top 5 or 10.

The accuracy comes from reading the query and the document together, rather than comparing two vectors that were computed separately.

## Bi-encoder against cross-encoder

This distinction is the whole idea, and interviewers ask for it by name.

| | Bi-encoder (retrieval) | Cross-encoder (reranking) |
|---|---|---|
| Input | Query and document encoded separately | Query and document encoded together |
| Comparison | Dot product of two vectors | One model pass per pair |
| Precompute | Document vectors, offline | Nothing, every pair is new work |
| Speed | Millions of items in milliseconds | Tens of items in tens of milliseconds |
| Accuracy | Good | Much better |

A cross-encoder sees the interaction between the query words and the document words. A bi-encoder cannot, because the document was encoded before the query existed. That is exactly the same trade as the one in [two-tower retrieval](two-tower-retrieval.md).

## The problem it solves

Vector search returns things that are broadly about the topic. The user asked a specific question. The passage that answers it is often at position 7, not position 1.

Two failures follow from skipping the rerank step. The prompt fills with near-misses, so the model has to guess. And the good passage falls outside the top 5 that fit the context budget, so the answer is wrong for a retrieval reason.

## How it works

```
query --> [retrieve 100] --> [rerank 100 pairs] --> [keep top 8] --> [generate]
```

Three kinds of reranker, in increasing cost:

- **Cross-encoder.** A small model trained for relevance scoring. The default.
- **LLM reranker.** Ask a language model to score or order the candidates. More flexible, far more expensive, and it needs the same care as any [judge](../evals/llm-as-judge.md).
- **Learned ranker with features.** In recommendation systems this stage is a full ranking model with user and item features, not a text relevance model. See [multi-stage ranking](multi-stage-ranking.md).

## When to use it

- Retrieval recall is good and the top few results are still wrong. That is exactly the symptom this fixes.
- The context budget is tight, so only a few passages can be sent.

Skip it when latency has no room, or when the candidate list is already short and accurate. Measure first: if precision at 5 is already high, a reranker costs latency and buys nothing.

## Trade-offs

| Pro | Con |
|-----|-----|
| Large precision gain at the top of the list | Adds 50 to 200 ms to every request |
| Needs no change to the index | Another model to serve, monitor, and version |
| Works with any retrieval source, including hybrid | Cost scales with the number of candidates reranked |
| Can be distilled into a smaller, faster model | A reranker trained on the wrong domain can be worse than nothing |

## Numbers to know

- Rerank 50 to 100 candidates, keep 5 to 10.
- Cross-encoder latency: roughly 1 to 4 ms per pair on a GPU, batched. That is 50 to 200 ms for 50 candidates.
- The precision gain at the top of the list is often the single largest quality jump in a RAG system.
- Doubling the reranked candidate count doubles the latency and rarely doubles the gain. Find the point where it flattens.

## How to talk about it in an interview

**The sentence to say.** "Retrieval gives me 100 candidates by vector and keyword. A cross-encoder reranks them, because it reads the query and the passage together, and I only send the top 8 to the model. That is 150 ms I am buying precision with."

Expect these follow-ups:

- Why not use the cross-encoder for retrieval too. Because it cannot precompute anything, so it would run millions of times per request.
- How you decide how many candidates to rerank.
- How you measure the reranker separately from the retriever. See [evals](../evals/).

## Go deeper

The full lesson on hybrid search and reranking: [Grokking the AI System Design Interview](https://www.designgurus.io/course/grokking-the-ai-system-design-interview).
