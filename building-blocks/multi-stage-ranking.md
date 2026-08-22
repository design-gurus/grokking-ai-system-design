# Multi-stage ranking

> A funnel where each stage sees fewer items and spends more compute on each one.

## What it is

Ranking systems do not score every item with the best model. They run a sequence of stages. Each stage is more accurate and more expensive per item than the one before, and each one hands a shorter list to the next.

## The problem it solves

The best model costs too much to run over the whole catalog. The cheapest model is not good enough to pick the final order. A funnel gets both: wide coverage at the top, high accuracy at the bottom.

## How it works

```
10,000,000 items
   --> [retrieval]        ~500 left    cheap, dot product only
   --> [ranking]          ~100 left    a real model, many features
   --> [reranking]         ~10 shown   heavy model, list-aware
   --> [rules]                         diversity, safety, business logic
```

| Stage | Items in | Cost per item | Job |
|-------|----------|---------------|-----|
| Retrieval | Millions | Microseconds | Lose nothing good |
| Ranking | Hundreds | A millisecond | Order them well |
| Reranking | Tens | Tens of milliseconds | Fix the top of the list |
| Rules | Tens | Free | Enforce what the model must not decide |

## The rule that governs the funnel

**Recall matters early, precision matters late.**

An item dropped in retrieval can never be recovered, no matter how good the ranker is. So retrieval is tuned for recall, and it is measured against a brute-force baseline.

The last stage decides what the user sees, so it is tuned for precision at the top few positions. A retrieval stage judged by precision, or a final stage judged by recall, is a design that will be questioned.

## The last stage is not a model

Diversity, safety, and business rules go after the model, not into it.

- **Diversity.** Five posts from one author is a worse feed than five from five, even when the model scores them highest. This is enforced with a rule, not learned.
- **Safety.** A harmful content classifier is a hard filter. Its output must not be a feature the ranker can trade away for engagement.
- **Business logic.** Sponsored placement, inventory, and contractual rules live here, where they can be audited and changed without retraining.

## When to use it

- The catalog is far larger than what one request can score.
- You need both semantic breadth and fine-grained ordering.

Skip stages when you can. A system with 5,000 items does not need retrieval, and a system with no list-level effects does not need reranking. Every stage adds latency, code, and one more place for quality to leak.

## Trade-offs

| Pro | Con |
|-----|-----|
| Scores millions of items within a latency budget | Each boundary is a place where good items are lost |
| Each stage can be improved on its own | Offline metrics per stage do not add up to end quality |
| Cheap stages absorb the volume | More stages means more models to train and monitor |
| Rules stay outside the model, so they are auditable | Rules and model can fight each other in ways that are hard to debug |

## Numbers to know

- Common funnel widths: millions to 500 to 100 to 10.
- A full request budget of 300 ms splits roughly 20 ms retrieval, 60 ms ranking, 50 ms reranking, and the rest as slack.
- Doubling the candidate count rarely doubles quality. Measure the recall gain before paying for it.

## How to talk about it in an interview

The signal is naming the metric for each stage separately.

**The sentence to say.** "Retrieval is tuned for recall at 500, because anything dropped there is gone. Ranking is tuned for AUC over those candidates. Diversity and safety are rules after the model, not features inside it."

Expect these follow-ups:

- How you would decide the width of each stage.
- What happens when the safety filter removes most of the list.
- How you attribute an end-to-end quality drop to one stage.

## Go deeper

- Question that uses this end to end: [design feed ranking](../questions/design-feed-ranking.md)
- Course: [Grokking the AI System Design Interview](https://www.designgurus.io/course/grokking-the-ai-system-design-interview)
