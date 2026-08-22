# Design model monitoring

> Observability for systems that fail without raising an error, where the signal proving something broke arrives weeks after it broke.

**Family:** Platform
**Also asked as:** design ML observability, design a drift monitoring system.

## 1. Frame the problem

Ordinary monitoring answers "is the service up". This platform answers "is the service still right".

Those are different questions, because a model degrades with no exception, no error rate change, and no latency change. Every request succeeds. The answers just get worse.

The platform's job: watch hundreds of models across many teams, detect quality problems as early as the available signals allow, and point at the cause.

Scale to assume: 300 models, 50,000 predictions per second in total, labels that arrive anywhere from seconds to 90 days later.

## 2. Define success

| Metric type | Metric | Target |
|-------------|--------|--------|
| Detection | Time from a real degradation to an alert | Hours, not weeks |
| Precision | Share of alerts that were a real problem | High enough that teams still read them |
| Coverage | Share of production models with monitoring configured | Total, by default |
| Cost | Storage and compute per million predictions monitored | Bounded by sampling |

Alert precision is the metric that decides whether the platform is used. A monitor that fires weekly on noise is turned off within a month, and then nothing is monitored at all.

## 3. Map the data

Four streams, in increasing order of delay and value.

| Stream | Available | Tells you |
|--------|-----------|-----------|
| Feature values sent to the model | Immediately | Input drift, and broken pipelines |
| Predictions | Immediately | Output drift, threshold shifts |
| Business events downstream | Minutes to hours | Whether users act on the output |
| Labels | Days to months | The actual truth |

Log a sample of prediction requests: the feature values used, the prediction, the model version, and a request id that later joins to the label. That join key is the whole design. Without it you can never connect a prediction to its outcome.

Sample rather than logging everything. A few percent of traffic is enough for distribution monitoring, with full logging for errors and for a small random slice used in evaluation.

## 4. The core design choice

**Monitor the earliest signal, not the best one.**

Labels are the truth and they are far too slow to alert on. So the monitoring ladder runs in the other direction, from immediate and weak to delayed and strong:

1. **Input distributions.** Compare live feature distributions to a reference window. No labels needed, and this catches the most common real incident, which is a feature pipeline that started returning nulls or zeros.
2. **Prediction distributions.** Score histograms and class balance over time.
3. **Proxy outcomes.** Click rate, acceptance rate, override rate. Available in hours.
4. **Labeled accuracy.** The truth, computed when labels land, reported as a trailing number.

**Slice everything.** An aggregate is where regressions hide. A model can hold steady overall while collapsing for one country, one device, or one customer tier. Define the slices per model up front, and alert per slice.

## 5. The serving path

```
[serving] --> [sampled log: features, prediction, version, request id]
          --> [stream aggregator: distributions per feature, per slice, per window]
          --> [comparison against reference] --> [alerts]
[labels]  --> [join by request id] --> [trailing accuracy, per slice]
```

The comparison is a scheduled job over aggregated statistics, not a scan of raw logs. Store the aggregates long-term and the raw samples for a short window, which keeps the cost bounded.

Alerts route to the model's owning team with the diagnosis attached: which feature moved, which slice, and how far. An alert that says only "drift detected" creates work rather than saving it.

## 6. Keep it working

- **Reference windows go stale.** A comparison against a fixed old window fires constantly after any legitimate change. Update the reference when a new model version ships.
- **Seasonality is not drift.** Weekend traffic differs from weekday traffic. Compare like periods, or the platform alerts every Saturday for no reason.
- **Distinguish the causes.** Input drift with a feature at exactly zero is a broken pipeline. Input drift with a smooth shift is the world moving. The first is an incident, the second is a retraining trigger. See [drift detection and retraining](../building-blocks/drift-detection-and-retraining.md).
- **Monitoring must be on by default.** Anything opt-in covers the models whose owners were already careful.

## 7. Count the costs

Storage dominates: features and predictions for 50,000 requests per second adds up quickly. Sampling and short retention on raw logs, with long retention on aggregates, is the standard shape.

Levers: sample rates per model by importance, aggregate at write time rather than storing raw, and keep only the slices that are actually alerted on.

## Common follow-ups

- **How do you detect a problem when labels take 60 days?** Input and prediction distributions, plus proxy outcomes. Say the ladder.
- **What is the most common alert in practice?** A broken feature pipeline, not genuine drift.
- **How do you avoid alert fatigue?** Slice-level thresholds, seasonality-aware comparison, and a diagnosis attached to every alert.
- **What triggers a retrain?** A sustained drift signal plus a quality drop, gated by the evaluation suite before anything deploys.

## Common mistakes

- Monitoring only latency and error rate.
- Waiting for labels to detect anything.
- Aggregate-only metrics, so per-slice failures are invisible.
- Logging everything, then turning it off when the bill arrives.

## Go deeper

- Related: [drift detection and retraining](../building-blocks/drift-detection-and-retraining.md), [evals](../evals/)
- The full worked solution: [Grokking the AI System Design Interview](https://www.designgurus.io/course/grokking-the-ai-system-design-interview)
