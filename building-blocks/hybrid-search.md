# Hybrid search

> Running keyword search and vector search together, because each one misses exactly what the other finds.

## What it is

Hybrid search sends the query to two retrieval systems and merges the results. One matches words. The other matches meaning.

Keyword search scores documents by term overlap, usually with BM25, a formula that rewards rare words and short documents. Vector search compares [embeddings](embeddings.md).

## The problem it solves

Each method fails in a way the other does not.

| Query | Keyword search | Vector search |
|-------|----------------|---------------|
| "how do I cancel my plan" | Misses a page titled "Ending your subscription" | Finds it |
| "error ORA-01555" | Finds the exact page | Returns pages about database errors in general |
| "SKU 4471-B" | Finds it | Returns visually similar product text |
| "why is my bill higher this month" | Matches only on shared words | Finds the billing policy page |

A support corpus and an enterprise wiki contain both kinds of query, from the same user, minutes apart. Picking one retrieval method means accepting a known failure class.

## How it works

```
query --> [BM25 index]   --> ranked list A
      --> [vector index] --> ranked list B
                          --> merge --> top k --> [rerank] --> answer
```

The merge is the interesting part, and there are two ways to do it.

**Reciprocal rank fusion** ignores the scores and uses only the positions. Each document gets a score of 1 divided by (a constant plus its rank) in each list, and the two are added. It is the default because it needs no tuning and no calibration.

**Weighted score fusion** normalizes both score scales and adds them with a weight. It can beat fusion by rank, and it needs work: BM25 scores are unbounded and corpus-dependent, cosine scores sit between minus one and one. Comparing them raw is meaningless.

Start with reciprocal rank fusion. Move to weighted scores only when you have a labeled set to tune against.

## When to use it

- Any corpus containing identifiers, codes, names, or version numbers, which is most real corpora.
- Any product where users type both natural questions and exact strings.

Skip it when the query is never text the user typed, for example pure image similarity or a recommendation feed with no search box.

## Trade-offs

| Pro | Con |
|-----|-----|
| Covers both exact terms and meaning | Two indexes to build, sync, and monitor |
| Large recall gain for little tuning | Two systems can disagree about which documents exist |
| Fusion by rank needs no calibration | Latency is the slower of the two, plus the merge |
| Degrades well: one index down still returns results | Filters and permissions must be applied consistently in both |

## The consistency trap

Both indexes must apply the same permission filter and see the same deletions. If a document is removed from one index and not the other, it comes back through the surviving path. Delete propagation is a correctness requirement, and it is worth an alert.

## Numbers to know

- Hybrid typically adds several points of recall over either method alone. Measure it on your own labeled queries, because the gain depends on how many exact-term queries you get.
- Latency: run both in parallel, so the cost is the slower one plus a few milliseconds of merging.
- Retrieve 50 to 100 candidates from each side before merging, then cut to 5 to 10 after [reranking](reranking.md).

## How to talk about it in an interview

**The sentence to say.** "I would run BM25 and vector search in parallel and merge with reciprocal rank fusion. Users search for error codes and ticket numbers as often as they ask questions, and pure vector search misses exact strings."

Expect these follow-ups:

- How you merge two score scales. Say why rank fusion avoids the problem.
- What happens when one index is stale.
- How you would tell whether hybrid actually helped.

## Go deeper

The full lesson on hybrid search and reranking: [Grokking the AI System Design Interview](https://www.designgurus.io/course/grokking-the-ai-system-design-interview).
