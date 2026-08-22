# Design ad click prediction

> Predict the probability that a user clicks an ad, in under 50 ms, where the probability itself is the price.

**Family:** Classic ML
**Also asked as:** design an ads ranking system, design pCTR, design the ad auction.

## 1. Frame the problem

Predict the click probability for a user, an ad, and a context. The auction then ranks by expected value, usually bid multiplied by predicted click rate.

That multiplication is what makes this question different from every other ranking problem. **The predicted probability is used as a number, not as an ordering.** A model that ranks perfectly but predicts 0.4 where the truth is 0.04 charges advertisers ten times too much and destroys the marketplace.

Scale to assume: 100,000 ad requests per second at peak, millions of active ads, a 50 ms budget for the whole auction.

## 2. Define success

| Metric type | Metric | Target |
|-------------|--------|--------|
| Offline | Log loss, or normalized entropy against the base rate | Lower than the current model |
| Offline | Calibration: predicted rate against observed rate, by bucket | Within a few percent across buckets |
| Online | Revenue per thousand requests, advertiser return | Up, with advertiser return not down |
| Guardrail | p99 latency, ad load, user complaint rate | Latency under 50 ms |

**Report calibration, not just AUC.** AUC is invariant to any monotonic rescaling of the scores, so a badly calibrated model can have an excellent AUC. Saying this sentence is the single strongest signal in this question.

## 3. Map the data

**Labels.** An impression with no click is a negative, and clicks are rare, often around 1 percent. Conversions are rarer still and arrive hours or days later, which matters if you optimize for conversion rather than clicks.

**Features are enormous and sparse.** User id, ad id, campaign, advertiser, publisher, placement, device, hour, and every cross of them. Hundreds of millions of distinct values.

That shape drives two decisions:

- **Hashing.** Map high-cardinality values into a fixed space. Collisions are accepted in exchange for a bounded table size.
- **Embedding tables.** Each id gets a small learned vector. These tables dominate memory, often far more than the rest of the model.

Real-time counters matter too: this ad's click rate in the last hour, this user's ad fatigue today. They need a streaming pipeline, not a daily batch. See [feature stores](../building-blocks/feature-stores.md).

## 4. Pick the model

1. **Logistic regression on hashed features.** Trains fast, serves in microseconds, and is naturally calibrated. Still a real baseline at scale.
2. **Gradient-boosted trees feeding a linear layer**, the classic hybrid.
3. **Deep model with embedding tables.** More accurate, and it needs recalibration and a much larger memory footprint.

Whatever the model, add a calibration layer on top, fitted on recent data. Isotonic regression or Platt scaling. Refit it often, because the base rate moves with traffic mix.

## 5. The serving path

Budget: 50 ms for retrieval, scoring, and the auction.

```
[request] --> [targeting filters: budget, geo, eligibility]  ~thousands of ads
          --> [light model, ~500 ads]
          --> [full model, ~100 ads, 15 ms]
          --> [auction: bid x pCTR, reserve price]
          --> [pacing and frequency caps] --> [ads]
```

Budget pacing sits outside the model. An advertiser with a daily budget must not spend it by 9 in the morning, and that is a control loop, not a prediction.

Failure policy matters here. If the model times out, serve a lower-value fallback rather than an empty slot, and record that the auction was degraded.

## 6. Keep it working

- **Recalibrate continuously.** Calibration drifts faster than ranking quality.
- **New ads have no history.** They need exploration, and exploration costs real money, so bound it: a small budget of impressions per new ad, then let the data decide.
- **Watch the feedback loop.** Ads the model does not show get no data and stay unshown.
- **Segment the metrics.** Overall revenue can rise while a whole advertiser category collapses.

## 7. Count the costs

Serving cost is dominated by scoring 100,000 requests per second against hundreds of candidates. Memory cost is dominated by embedding tables, which can reach hundreds of gigabytes and must be sharded across machines.

Levers: shrink the candidate set with better targeting filters, quantize embedding tables, and cache scores for repeated user and ad pairs within a short window.

## Common follow-ups

- **Why is calibration more important than AUC here?** Because the probability sets the price.
- **A brand new advertiser.** Content and category features, plus a bounded exploration budget.
- **Delayed conversions.** Train on clicks now, attribute conversions later, and correct with a delay model.
- **How do you stop showing the same ad repeatedly?** Frequency caps in serving, not a model feature.

## Common mistakes

- Optimizing AUC and never mentioning calibration.
- Forgetting the auction, and designing only the model.
- Ignoring the memory cost of embedding tables.
- Putting budget pacing inside the model.

## Go deeper

- Written for a general system design round, architecture first: [Design an ad click aggregator](https://github.com/design-gurus/grokking-system-design/blob/main/questions/design-ad-click-aggregator.md).
- The full worked solution: [Grokking the AI System Design Interview](https://www.designgurus.io/course/grokking-the-ai-system-design-interview).
