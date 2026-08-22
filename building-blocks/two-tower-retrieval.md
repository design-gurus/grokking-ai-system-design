# Two-tower retrieval

> Narrowing millions of candidates to a few hundred in 20 ms, by precomputing one side of the comparison.

## What it is

Two-tower retrieval uses two encoder models. One turns the user and their context into a vector. The other turns an item into a vector. Similarity is the dot product of the two.

The towers are separate on purpose. Item vectors can be computed in advance, because nothing about them depends on the request.

## The problem it solves

A ranking model that scores a user and an item together is accurate and slow. Running it over 10 million items per request is impossible inside a 300 ms budget.

Retrieval solves the volume problem, not the accuracy problem. It has one job: get the few hundred items worth scoring properly, and lose as few good ones as possible along the way.

## How it works

```
offline:  every item --> [item tower] --> vector --> [vector index]

online:   user + context --> [user tower] --> vector
                                          --> index lookup --> top 500 candidates
```

Item vectors are computed in a batch job and loaded into a [vector index](vector-indexes.md). At request time only the user tower runs, which is one small forward pass. The rest is an approximate nearest neighbor lookup.

**Training uses in-batch negatives.** For each positive pair, the other items in the training batch act as negatives. This is cheap and it is why the method scales. Adding harder negatives, items the model currently ranks too highly, improves quality more than adding more easy ones.

## The constraint that defines it

The two towers never see each other before the dot product. So the model cannot use features that combine the user and the item, like "how many times this user viewed this author this week".

That constraint is the price of precomputation. Interaction features belong in the ranking stage, where only a few hundred items are left. See [multi-stage ranking](multi-stage-ranking.md).

## When to use it

- The candidate set is too large to score item by item.
- Item vectors are stable enough to precompute, and refresh in hours rather than seconds.
- You already need semantic matching, so [embeddings](embeddings.md) exist anyway.

Skip it when the catalog is small. Scoring 2,000 items directly with a real ranker is simpler and better.

## Trade-offs

| Pro | Con |
|-----|-----|
| Millions of candidates in tens of milliseconds | No user-item interaction features |
| Item vectors are computed once, offline | New items have no vector until the next batch run |
| One index serves many surfaces | Retrieval quality is capped by the tower's simplicity |
| Trains on ordinary interaction logs | Needs negative sampling done well, which is easy to get wrong |

## Retrieval is not one source

Production retrieval blends several sources into the candidate pool, and saying so is a senior signal:

- The two-tower model, for personalized breadth.
- Simple rules: recent items from accounts the user follows.
- Popular or trending items, which cover cold-start users.
- A small exploration pool, so new items get a chance to collect data.

## Numbers to know

- Typical funnel: millions of items to about 500 candidates in one lookup.
- User tower inference: a few milliseconds. Index lookup: 5 to 50 ms.
- Item vectors are usually refreshed hourly or daily.
- Recall at 500 against a brute-force baseline is the metric that matters. Precision is the ranker's job.

## How to talk about it in an interview

The signal is that you know which stage each feature belongs to.

**The sentence to say.** "Retrieval is a two-tower model, so item vectors are precomputed and the request only runs the user tower plus an index lookup. Interaction features come in at the ranking stage, over 500 candidates rather than 10 million."

Expect these follow-ups:

- How you pick negatives during training.
- What happens to an item posted one minute ago.
- How you measure retrieval separately from ranking.

## Go deeper

The full lesson on embeddings and two-tower retrieval: [Grokking the AI System Design Interview](https://www.designgurus.io/course/grokking-the-ai-system-design-interview).
