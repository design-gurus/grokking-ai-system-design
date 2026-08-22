# Design visual search

> Find catalog items that look like the photo a user just took, when the photo is badly lit, at an angle, and contains three other objects.

**Family:** Classic ML
**Also asked as:** design search by image, design Pinterest Lens, design "shop the look".

## 1. Frame the problem

Given a query image, return visually similar items from a catalog. The user's intent is usually "sell me this or something like it".

The task is retrieval in an embedding space, but the framing question is what similar means. Same product, same style, or same category? A shopper photographing a chair may want that exact chair, or any chair in that style, and the two are different systems.

**The hardest part is a distribution mismatch.** Catalog images are clean, lit, and centered. User photos are none of those. A model trained only on catalog images performs far worse on real queries than the offline number suggests.

Scale to assume: a 20 million item catalog, 5 million visual searches a day.

## 2. Define success

| Metric type | Metric | Target |
|-------------|--------|--------|
| Offline | Recall at 10 on labeled query and item pairs, using real user photos | Stated per launch |
| Online | Click-through and purchase rate from visual results | Comparable to text search |
| Guardrail | p99 latency, share of searches with no acceptable result | Latency under 500 ms |

**The offline set must use user photos, not catalog images.** A set built from catalog crops measures the easy case and hides the failure you actually have.

## 3. Map the data

**Labels for similarity come from behavior, not from annotators.** Items purchased after the same visual search, or items co-viewed in one session, are cheap positives. Human labeling covers the evaluation set, where quality matters more than volume.

**Training data needs the messy distribution.** Two ways to get it. Collect real user query photos with the item they eventually clicked. And augment catalog images with crops, rotations, lighting changes, and new backgrounds, so the model learns to ignore them.

**Metadata rides along with the vector.** Category, color, price, brand, and availability are needed for filtering and reranking, and they must be in the index rather than fetched afterwards. See [vector indexes](../building-blocks/vector-indexes.md).

## 4. Pick the model

1. **A pretrained vision encoder, used as is.** The baseline, and it works surprisingly well for category-level similarity.
2. **The same encoder fine-tuned with metric learning** on your own positive pairs, which is where the real gain is. Train it so that two images of the same product are closer than two images of different products in the same category.
3. **Detection first, then embedding.** Real photos contain several objects. Detect the regions, let the user choose or pick the largest salient one, then embed the crop rather than the whole image.

A joint image and text model is worth naming, because it lets one index serve "photos like this" and "red canvas sneakers" together.

## 5. The serving path

Budget: 500 ms, more generous than text search because users expect an upload to take a moment.

```
[photo] --> [resize, detect objects]  100 ms
        --> [embed the selected crop]  50 ms
        --> [ANN search with filters: in stock, region]  30 ms
        --> [rerank with attributes and business signals]  50 ms
        --> [results]
```

Filters go inside the index query. Retrieving 100 visually perfect matches and then dropping the 95 that are out of stock leaves a bad result page. It is the same trap as permission filtering in a document system.

Catalog embeddings are precomputed in a batch job when items are added or their images change. Only the query image is embedded live.

## 6. Keep it working

- **New items must be searchable quickly.** Embedding on upload, with a minutes-level target, not a nightly batch.
- **An encoder change means re-embedding 20 million items.** Build the new index alongside, compare recall, then cut over. Version the encoder in the index metadata.
- **Watch seasonal drift.** Query content changes with the seasons, and the catalog turns over.
- **Sample failed searches.** Sessions with no click are the dataset that tells you which query types are broken.

## 7. Count the costs

Two costs: embedding 5 million query images a day on GPUs, and holding 20 million catalog vectors in memory.

At 768 dimensions and 4 bytes per float, 20 million vectors is about 60 GB of raw vectors before index overhead. That is a decision point: one large machine, sharding, or compression with rescoring.

Levers: smaller embedding dimensions, quantized vectors with exact rescoring at the end, and caching results for repeated or near-identical query images.

## Common follow-ups

- **The photo has several objects.** Detect and crop, and let the user pick the region.
- **Exact match or similar items?** Ask which the product wants. Exact needs tighter thresholds and often a second verification step.
- **Combining image and text.** A shared embedding space, or filter by the text and rank by the image.
- **How do you evaluate?** Recall at k on real user photos, plus click and purchase rate online.

## Common mistakes

- Training and evaluating only on clean catalog images.
- Embedding the whole photo when it contains several objects.
- Filtering stock and region after retrieval.
- No plan for the re-embedding project that an encoder upgrade requires.

## Go deeper

- Related: [embeddings](../building-blocks/embeddings.md), [vector indexes](../building-blocks/vector-indexes.md)
- The full worked solution: [Grokking the AI System Design Interview](https://www.designgurus.io/course/grokking-the-ai-system-design-interview)
