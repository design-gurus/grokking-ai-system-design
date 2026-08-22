# Design feed ranking

> Choose the order of posts in a social feed, for hundreds of millions of users, in under 300 ms.

**Family:** Classic ML
**Also asked as:** design the Facebook news feed, design the Twitter timeline, design the TikTok For You page.

## 1. Frame the problem

The product ask is "show me a good feed". The machine learning task is narrower. Given a user and a candidate post, predict the probability of an interaction in this session. Then sort by a score built from those probabilities.

This question contains two systems. Retrieval picks a few hundred candidates out of millions. Ranking orders them. Interviewers want to see you separate the two, because they have different constraints.

Scale to assume: 300 million daily users, 20 sessions a day, 50 posts scored per session at minimum. That is roughly 3 million ranking requests per second at peak if you score on read, which immediately shapes the design.

## 2. Define success

| Metric type | Metric | Target |
|-------------|--------|--------|
| Offline | AUC or normalized entropy on held-out interactions | Beat the current model by a stated margin |
| Online | Session time, or daily active users returning | Up 1 to 3 percent, measured in an A/B test |
| Guardrail | p99 latency, reported content rate, creator diversity | Latency under 300 ms, reports flat or lower |

Say why you are not optimizing clicks alone. A click model learns to promote outrage and clickbait, which raises clicks and lowers retention. The usual fix is a weighted combination of several predicted actions: click, long dwell, like, share, hide, report. Negative actions carry negative weight.

**The weights are a product decision, not a model decision.** Saying that out loud is a senior signal.

## 3. Map the data

**Labels** come from logged user behavior. An impression with no interaction is a negative. Positives are clicks, dwell over some threshold, likes, and shares. Labels arrive in seconds, which is why feed models retrain often.

Two traps to name:

- **Position bias.** Users click the top item because it is at the top. Train on that and the model learns position, not quality. The fix is to include position as a feature during training, then set it to a constant at serving time. Logging a small amount of randomized ordering also works.
- **Feedback loops.** The model only ever sees interactions on the posts it chose to show. Posts it never showed have no data. A small share of exploration traffic keeps some of those posts in the training data.

**Features**, in three groups:

| Group | Examples | Freshness |
|-------|----------|-----------|
| User | Long-term interests, language, session count, recent topics | Batch daily, plus a streaming session feature |
| Item | Age of the post, author, media type, early engagement rate | Streaming, because a post's engagement moves in minutes |
| Interaction | Has the user seen this author before, affinity score, time since last view | Computed at request time or in a [feature store](../building-blocks/feature-stores.md) |

Early engagement rate is the highest-value feature and the most dangerous. It is computed from the last few minutes of traffic, so the pipeline that produces it must be a stream job. The model must also handle a missing value, because a brand new post has none.

## 4. Pick the model

Retrieval: a two-tower model. One tower embeds the user, one embeds the post, and similarity is a dot product. User embeddings are computed at request time, post embeddings are precomputed, so retrieval is a [vector index](../building-blocks/vector-indexes.md) lookup. Add non-model sources next to it: recent posts from followed accounts, trending items, and a small exploration pool.

Ranking: start with gradient-boosted trees on 50 to 100 features. They train in minutes, are easy to debug, and give you a number the next model has to beat. Upgrade to a neural ranker with multi-task heads, one head per predicted action, sharing a lower layer. That is where the weighted combination of actions comes from.

Do not open with the neural model. Open with the baseline, then say what the upgrade buys and what it costs in serving latency.

## 5. The serving path

Budget: 300 ms end to end for the feed request.

```
[request] --> [retrieval: ~500 candidates, 20 ms]
          --> [feature fetch: batched, 30 ms]
          --> [ranker: 500 scores, 60 ms]
          --> [rules and diversity: 10 ms]
          --> [response]
```

Score on read or precompute? Precomputing a feed for every user wastes work on users who do not log in, and it goes stale within minutes. Scoring on read is fresh and expensive. The common answer is a hybrid: precompute candidates for active users, score on read.

Degradation matters here. If the ranker is unavailable, serve the reverse-chronological list from followed accounts. A worse feed is acceptable. An error page is not.

## 6. Keep it working

- **Retrain daily or faster.** Feed data moves quickly, and a week-old model measurably underperforms.
- **Monitor feature drift** before quality drops. A feature pipeline that silently returns zeros is the most common outage, and the model will not raise an error, it will just rank badly.
- **Every launch is an experiment.** Ship behind an A/B test with the guardrails from step 2. Hold back a small share of users on the old model long enough to see retention effects, not just session effects.
- **Watch the ecosystem, not just the user.** If the model gives all reach to 100 creators, most other creators stop posting. Creator diversity belongs in the guardrails.

## 7. Count the costs

The cost is dominated by ranking inference. 3 million requests per second, 500 candidates each, is 1.5 billion scores per second. That number is the reason the ranker must be small and the retrieval stage must be aggressive.

Levers: cut candidates from 500 to 200 for low-activity users. Cache scores for a few minutes per user. Use a smaller model for users with little history.

## Common follow-ups

- **Cold start for a new user.** Popularity and demographic defaults, plus fast adaptation from the first session's clicks.
- **Cold start for a new post.** Give it a small share of exploration traffic and use content features until engagement features exist.
- **Why not just chronological?** It is a real option and a fair guardrail baseline. Say what ranking buys, and admit that it costs the user's sense of control.
- **How do you stop the model promoting harmful content?** A separate classifier runs before ranking, and its output is a hard filter, not a feature.

## Common mistakes

- Describing one giant model that both retrieves and ranks.
- Optimizing a single click metric with no negative actions.
- Forgetting position bias, then claiming a high offline AUC means the model is good.
- Ignoring the latency budget, then proposing a large neural ranker over 5,000 candidates.

## Go deeper

The full worked solution, with the capstone version of this question: [Grokking the AI System Design Interview](https://www.designgurus.io/course/grokking-the-ai-system-design-interview).
