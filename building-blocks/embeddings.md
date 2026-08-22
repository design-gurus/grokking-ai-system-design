# Embeddings

> Turning text, images, or users into lists of numbers, so that similar things end up close together.

## What it is

An embedding is a list of numbers that represents one item. A model produces it. The useful property is geometric: two items with similar meaning get vectors that point in a similar direction.

Once items are vectors, "find things like this" becomes arithmetic, and arithmetic is fast.

## The problem it solves

Exact matching cannot answer "find documents about cancelling a subscription" when the document says "ending your plan". The two share almost no words.

Embeddings put both near each other, because the model that produced them was trained on text where those phrases appear in the same contexts. That is the whole trick, and it powers search, recommendation, deduplication, and clustering.

## How it works

```
[text or image] --> [encoder model] --> [vector of 768 to 1536 floats]
```

Similarity between two vectors is one number. Two measures cover almost everything:

- **Cosine similarity** compares direction only, ignoring length. It is the default for text.
- **Dot product** compares direction and length together. Used when length carries meaning, for example item popularity baked into the vector.

If vectors are normalized to unit length, the two are equivalent. Most systems normalize, which removes a whole class of confusing bugs.

## When to use it

- Retrieval where meaning matters more than exact words.
- Recommendation, where user and item live in the same space. See [two-tower retrieval](two-tower-retrieval.md).
- Grouping or deduplicating near-identical items.
- As features inside a larger model, for example a category embedding in a ranker.

Do not use them where exact matching is the requirement. Order numbers, error codes, and product SKUs are found by keyword search, not by similarity. That is why production retrieval runs [hybrid search](hybrid-search.md).

## The rules that prevent real bugs

**Never compare vectors from two different models.** Their spaces are unrelated, and the similarity numbers you get are meaningless rather than wrong-looking. Store the model name and version next to every vector.

**The same model must embed the query and the document.** A retrieval system that upgrades one side and not the other returns nonsense, and nothing in the pipeline will raise an error.

**Changing the embedding model means re-embedding everything.** For millions of documents this is a project with a cost and a schedule, not a config change. Build the new index alongside the old one, compare recall on a labeled set, then cut over.

## Trade-offs

| Pro | Con |
|-----|-----|
| Finds matches that share no words | Misses exact identifiers and rare terms |
| One vector serves search, grouping, and features | A model upgrade invalidates every stored vector |
| Cheap to compute and store per item | Storage grows fast at scale, 3 to 6 KB per vector |
| Works across languages with a multilingual model | Quality varies by domain, and must be measured on your data |

## Numbers to know

- Common dimensions: 384, 768, or 1536 floats per vector.
- Storage: dimensions times 4 bytes, so about 3 KB at 768 and 6 KB at 1536.
- Embedding a page of text costs a fraction of a cent and takes tens of milliseconds.
- Smaller dimensions retrieve faster and store cheaper, at some quality cost. Measure before choosing.

## How to talk about it in an interview

The signal is that you treat the embedding model as a versioned dependency, not a utility function.

**The sentence to say.** "I would pick an embedding model by measuring recall on a few hundred labeled queries from our own data. The version goes in the index metadata, because changing it later means re-embedding the whole corpus."

Expect these follow-ups:

- Cosine or dot product, and why.
- What happens when you need to add a new language or a new document type.
- How you would evaluate one embedding model against another. See [evals](../evals/).

## Go deeper

The full lesson on embeddings and vector databases: [Grokking the AI System Design Interview](https://www.designgurus.io/course/grokking-the-ai-system-design-interview).
