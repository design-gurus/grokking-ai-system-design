# Design search ranking

> Order results for a query the user actually typed, which is the one case where intent is handed to you.

**Family:** Classic ML
**Also asked as:** design Google search ranking, design product search, design search relevance.

## 1. Frame the problem

The query is the signal. That makes this different from recommendation, where intent has to be inferred.

The task: given a query, a user, and a document, predict relevance, then order by it. Relevance has to be defined for the product. For a shop it is "would buy this". For a help center it is "answers the question". For a marketplace it is both, plus availability.

Scale to assume: 10 million documents, 5 million queries a day, a heavy head of repeated queries and a very long tail of rare ones.

## 2. Define success

| Metric type | Metric | Target |
|-------------|--------|--------|
| Offline | NDCG at 10 on human-judged query and document pairs | Stated per launch |
| Online | Click-through at position, query reformulation rate, zero-result rate | Reformulations down |
| Guardrail | p99 latency, share of queries with no result, spam in the top 10 | Latency under 200 ms |

Reformulation rate is the metric worth naming. A user who searches again with different words did not find what they wanted, and no click metric shows that.

Human relevance judgments are the offline set. They cost money, so you buy a few thousand query and document pairs, weighted toward the head and toward queries you are trying to fix.

## 3. Map the data

**Click logs are the volume, and they are biased.** Position bias is the largest effect: rank 1 gets clicked because it is rank 1. Correct for it with a click model that separates "was it examined" from "was it relevant", or by logging a small amount of randomized ordering.

Other signals: dwell time after the click, whether the user came back and clicked something else, add-to-cart, and purchase.

**Features** group into three:

- **Query features.** Length, language, whether it looks like an identifier, predicted category.
- **Document features.** Quality, freshness, popularity, price, stock.
- **Query and document features.** BM25 score, vector similarity, exact-title match, category match. This group carries the ranking.

## 4. Pick the model

1. **BM25 alone.** The baseline. It is strong, it is free, and a neural system that cannot beat it is not ready.
2. **Gradient-boosted ranker** over a few dozen features, trained on click data with position correction. This is where most production systems live.
3. **Neural reranking** with a cross-encoder on the top candidates. See [reranking](../building-blocks/reranking.md).

Retrieval is [hybrid search](../building-blocks/hybrid-search.md): an inverted index for exact terms and a vector index for meaning. Queries contain both product names and plain questions, so one method alone has a known failure class.

## 5. The serving path

Budget: 200 ms.

```
[query] --> [normalize, spell correct, expand]
        --> [retrieve: BM25 + vector, in parallel, ~1000 docs]
        --> [merge]
        --> [rank, ~200 docs, 40 ms]
        --> [rerank top 50 with a cross-encoder, 80 ms]
        --> [dedupe, diversity, filters] --> [results]
```

Query understanding at the front matters more than the ranker for tail queries. Spell correction, synonym expansion, and detecting that "iphone 14 case" has a product and an accessory type in it.

Cache aggressively. The head of the query distribution repeats constantly, and a cached result for a popular query is nearly free.

## 6. Keep it working

- **Zero-result queries are a work queue.** Each one is a gap in the index, the synonyms, or the understanding layer.
- **Index freshness has its own budget.** A new document should be searchable within minutes for news, hours for a catalog.
- **Watch the tail separately.** Head queries look fine while tail quality falls, and the tail is most of the distinct queries.
- **Retrain when the click distribution moves**, and re-judge the offline set when the product changes.

## 7. Count the costs

The reranker dominates. 5 million queries a day at 50 candidates is 250 million cross-encoder passes.

Levers: rerank fewer candidates, cache the head, and skip reranking for queries where the retrieval scores are already clearly separated.

## Common follow-ups

- **Tail queries with no clicks.** Lean on lexical matching and content features, because there is no behavior to learn from.
- **How would you compare two rankers cheaply?** Interleaving. It needs far less traffic than an A/B test. See [online experiments](../building-blocks/online-experiments.md).
- **Personalization.** Useful for ambiguous queries, harmful for precise ones. Apply it where the query is broad.
- **Spam and manipulation.** Sellers optimize against your ranker. Quality features and a separate spam classifier, not a ranker feature that can be traded away.

## Common mistakes

- Training on raw clicks with no position correction.
- Replacing keyword search with vector search, then failing on part numbers.
- Reporting NDCG on head queries only.
- No plan for zero-result queries.

## Go deeper

- Written for a general system design round, architecture first: [Design Google search](https://github.com/design-gurus/grokking-system-design/blob/main/questions/design-google-search.md).
- The full worked solution: [Grokking the AI System Design Interview](https://www.designgurus.io/course/grokking-the-ai-system-design-interview).
