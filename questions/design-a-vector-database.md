# Design a vector database

> A store whose primary query is "find the nearest vectors", where the answer is allowed to be approximate and the filter cannot be applied afterwards.

**Family:** Platform
**Also asked as:** design semantic search infrastructure, design an ANN service, design Pinecone.

## 1. Frame the problem

Store vectors with metadata, and answer nearest-neighbor queries with a filter, quickly.

The defining property is that exactness is negotiable. Scanning every vector is correct and far too slow, so the system returns approximate results and exposes recall as a tunable number. A design that does not mention recall has missed the point.

Scale to assume: 1 billion vectors at 768 dimensions, 10,000 queries per second, thousands of tenants, continuous inserts and deletes.

## 2. Define success

| Metric type | Metric | Target |
|-------------|--------|--------|
| Quality | Recall at 10 against a brute-force baseline on a sample | Stated per index, for example 0.95 |
| Service | p99 query latency | Tens of milliseconds |
| Freshness | Time from insert to searchable | Seconds |
| Efficiency | Vectors per gigabyte, queries per core | As high as recall allows |
| Isolation | One tenant's load or size does not affect another | Enforced |

Recall must be measured continuously, not once. It changes with the data distribution, with index parameters, and after compaction. A brute-force scan over a small sample gives the ground truth.

## 3. Map the data

Each record holds a vector, an id, and metadata used for filtering: tenant, permissions, category, timestamp.

Two storage paths:

- **The index**, holding vectors and the filterable metadata, in memory or on fast local disk.
- **The record store**, holding the original text or payload. Keep it out of the index, because loading the index into memory should not carry text with it.

**Metadata must be in the index**, not fetched afterwards. That is what makes filtered search correct, and it is the difference between a real design and a wrapper around a library.

## 4. The core design choice

**Index type, chosen by the binding constraint.**

| Family | Idea | Best when |
|--------|------|-----------|
| Graph, like HNSW | Walk a navigable graph toward the query | Default: fast and accurate, memory hungry |
| Cluster, like IVF | Search only the nearest clusters | Very large corpora, tunable memory |
| Quantized | Compress vectors into codes, rescore the top few exactly | Memory is the constraint |

At a billion vectors, memory decides. Raw vectors alone are roughly 3 TB at 768 dimensions, so compression plus exact rescoring is not optional.

**Sharding by vector, not by cluster.** Each shard holds a slice of the vectors and its own complete index. A query fans out to all shards, each returns its top k, and a coordinator merges them. This works because top k of the union equals the union of the per-shard top k, merged.

Sharding by content or by cluster breaks that property and creates hot shards.

**Filtered search has three correct approaches.** Push the filter into the graph walk. Use per-tenant indexes. Or pre-filter to a candidate set, when the filter is very selective. Filtering after retrieval is the one wrong answer, because a user whose filter excludes the top 100 gets nothing while matching records exist.

## 5. The serving path

```
[query vector + filter] --> [coordinator]
                        --> fan out to N shards, each returns top k
                        --> [merge, rescore exactly if compressed]
                        --> [attach payloads from the record store]
                        --> [results]
```

Query latency is the slowest shard, so shard sizes must stay balanced. Replicas per shard serve read throughput and survive a node failure.

## 6. Keep it working

- **Deletes are hard.** Graph indexes cannot cheaply remove a node, so deletions are tombstones and the index is compacted periodically. Until compaction, memory holds records nobody can retrieve.
- **Inserts degrade the graph.** Quality drifts as the data distribution moves, so periodic rebuilds are part of operations, not an incident.
- **Fresh writes need a small hot index** merged with the main one at query time. That is how you get seconds-level freshness without rebuilding.
- **An embedding model change is a full rebuild.** Support two index versions at once so a tenant can migrate without downtime.

## 7. Count the costs

Memory dominates. 1 billion vectors at 768 dimensions is about 3 TB raw, plus graph overhead. Compression at 8 or 16 times brings that into a fleet you can afford, at a recall cost you must measure.

Levers: lower dimensions, quantization with rescoring, tiering cold tenants to disk, and separating the payload store from the index.

## Common follow-ups

- **Why is filtering after search wrong?** Empty results while matching records exist, and a timing side channel.
- **How do you shard?** By vector, with a fan-out and merge. Explain why that preserves correctness.
- **What happens on delete?** Tombstone, then compaction.
- **How do you know recall is still 0.95?** Continuous sampling against a brute-force baseline.

## Common mistakes

- No mention of recall as a tunable number.
- Filtering after retrieval.
- Sharding by cluster, which creates hot shards.
- Storing payload text inside the index.

## Go deeper

- Building block: [vector indexes](../building-blocks/vector-indexes.md)
- Written for a general system design round, architecture first: [Design semantic search](https://github.com/design-gurus/grokking-system-design/blob/main/questions/design-semantic-search.md)
- The full worked solution: [Grokking the AI System Design Interview](https://www.designgurus.io/course/grokking-the-ai-system-design-interview)
