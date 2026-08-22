# Vector indexes

> Finding the most similar items among billions of vectors, in milliseconds, by accepting slightly wrong answers.

## What it is

An embedding is a list of numbers that represents an item, so that similar items have nearby numbers. A vector index is the data structure that finds the nearest vectors to a query vector quickly.

Exact nearest neighbor search compares the query to every vector. That is correct and far too slow: 10 million vectors at 768 dimensions is about 7.7 billion multiplications per query. Vector indexes use approximate nearest neighbor search, usually shortened to ANN. They return the right answers most of the time, in a few milliseconds.

## The problem it solves

Keyword search matches strings. It cannot find "how do I cancel my plan" inside a document titled "Ending your subscription", because the two share almost no words. Embeddings put those two near each other in vector space, and a vector index makes that lookup fast enough for a live request.

Every retrieval system needs this: semantic search, RAG, recommendation candidate generation, image search, and deduplication.

## How it works

Three index families cover almost every system.

| Family | Idea | Best when |
|--------|------|-----------|
| Graph based, like HNSW | Build a navigable graph, walk it greedily toward the query | The default: fast and accurate, memory hungry |
| Cluster based, like IVF | Group vectors into clusters, search only the nearest few | Very large corpora, tunable memory |
| Quantized, like product quantization | Compress vectors into codes, search the codes | Memory is the binding constraint |

These combine. A common production setup is clusters plus quantization, so the index fits in memory. A small exact rescoring step at the end fixes the errors compression introduced.

The one knob that matters is **recall against latency**. Every index lets you search harder for a better answer. Search harder and you find more of the true nearest neighbors, and each query takes longer. Recall at 10 of 0.95 means you found 95 percent of the true top 10.

## When to use it

- Similarity is the query, not equality or a range.
- The corpus is bigger than roughly 100,000 vectors. Below that, a brute-force scan in memory is simpler and fast enough, and it is exact.

Do not add a separate system too early. Several relational databases now hold vectors in a column next to the rest of the row. That removes an entire system, and keeps filters and permissions in one place.

## Trade-offs

| Pro | Con |
|-----|-----|
| Millisecond search over hundreds of millions of vectors | Answers are approximate, and recall is a number you must measure |
| Semantic matches that keyword search cannot find | Misses exact identifiers, error codes, and rare words, so hybrid search is usually needed |
| Scales by sharding on vector id | Filtered search is hard: filtering after retrieval can return nothing |
| Compression cuts memory by 4 to 32 times | Compression costs recall, and rescoring costs latency |

## The filtering trap

Most real queries have a filter: only this tenant, only documents this user may read, only items in stock. There are three ways to combine a filter with vector search, and only two are correct.

1. **Filter after search.** Retrieve the top 100, then drop what the user cannot see. If the user can see none of them, they get an empty answer while matching documents exist. This is a bug, and in a permissions setting it can also reveal that a document exists.
2. **Filter during search.** The index checks the filter while walking, so the top k is computed over allowed items only. This is what production systems do.
3. **Partition by the filter.** One index per tenant. Simple and exact, and it works well when the number of tenants is small.

Say this out loud in an enterprise question. It is the most common follow-up.

## Numbers to know

- A 768-dimension float vector is about 3 KB. 1536 dimensions is about 6 KB.
- 10 million vectors at 768 dimensions is roughly 30 GB, plus 20 to 50 percent index overhead for a graph index.
- Query latency: 5 to 50 ms, depending on recall target and corpus size.
- Rebuilding an index over 10 million vectors takes minutes to hours. An embedding model change is therefore a project, not a config change.

## How to talk about it in an interview

The signal is that you treat recall as a tunable number and know the filter problem exists.

**The sentence to say.** "I would use an approximate index with a recall target around 0.95, measured against a brute-force baseline on a sample. I would push permission filters into the index query, rather than filtering the results afterwards."

Expect these follow-ups:

- What happens when you change the embedding model. The answer is a full re-embed and a parallel index, then a cut over.
- How you handle updates and deletes. Graph indexes handle deletes with tombstones and need periodic compaction.
- Why you would add keyword search next to it.

## Go deeper

The full lesson on embeddings, indexes, and hybrid retrieval: [Grokking the AI System Design Interview](https://www.designgurus.io/course/grokking-the-ai-system-design-interview).
