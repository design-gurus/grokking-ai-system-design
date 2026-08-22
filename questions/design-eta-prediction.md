# Design ETA prediction

> Predict how long something will take, where being ten minutes late costs far more than being ten minutes early.

**Family:** Classic ML
**Also asked as:** design Uber ETA, design delivery time prediction, design Google Maps travel time.

## 1. Frame the problem

Predict the time between now and arrival. It is a regression problem, and the first design decision is what to predict.

**Predict a distribution, not a point.** Users are shown one number, but the system should produce a range, because the right number to display is not the average. Showing the 70th percentile means most deliveries arrive early, which is what people want.

The second decision is composition. A food delivery ETA is not one prediction. It is restaurant preparation time, plus courier assignment time, plus travel time, and each part has its own model and its own error.

Scale to assume: a food delivery service, 2 million orders a day, ETA recomputed several times per order.

## 2. Define success

| Metric type | Metric | Target |
|-------------|--------|--------|
| Offline | Mean absolute error, and quantile loss with a heavier penalty for late | MAE under a stated number of minutes |
| Online | On-time rate, order cancellations, support contacts about lateness | On-time up, contacts down |
| Guardrail | p99 latency, error on the worst segments, not just the average | Latency under 100 ms |

**The loss function is asymmetric and you should say so.** Ten minutes early is a mild annoyance. Ten minutes late is a cancelled order and a support contact. Train with a quantile loss rather than squared error, so the model learns that lateness costs more.

Watch the tail, not the mean. The average error can look excellent while 2 percent of orders are 30 minutes late, and those orders produce most of the complaints.

## 3. Map the data

**Labels are free and fast.** Every completed trip is a labeled example, available within an hour. That is the opposite of fraud, and it means this model can retrain daily.

**Features split by how fresh they must be:**

| Group | Examples | Freshness |
|-------|----------|-----------|
| Static | Distance, road network, historical segment speeds by hour and weekday | Daily batch |
| Contextual | Weather, day of week, whether a large event is on | Hourly |
| Real time | Current traffic on the route, restaurant's current queue, couriers available nearby | Seconds |

The real-time group carries most of the accuracy, and it is the hard engineering. A stream job maintains current segment speeds from live GPS traces, and the [feature store](../building-blocks/feature-stores.md) has to serve them in single-digit milliseconds.

**Watch for a selection bias.** You only observe travel times for routes that were actually taken. If dispatch avoids a road because the model says it is slow, you never learn that it got faster.

## 4. Pick the model

1. **Historical average per road segment, summed along the route.** The baseline. It is easily good enough to launch, and it makes the value of everything else measurable.
2. **Gradient-boosted trees** on route features plus real-time signals. This is where most production systems live, and they train in minutes.
3. **Segment-level models with a graph structure**, which capture how congestion on one road affects the next. A reasonable upgrade to name, not the starting point.

Predict the parts separately when the parts have different drivers. Preparation time depends on the restaurant and its current load. Travel time depends on the road network. One model for both learns a blurred average of two unrelated processes.

## 5. The serving path

Budget: 100 ms, and it is called many times per order.

```
[request] --> [route: distance and segments]  20 ms
          --> [feature fetch: segment speeds, restaurant load, courier supply]  25 ms
          --> [model per part]  20 ms
          --> [combine, add the safety buffer, round]  --> [ETA shown]
```

**Recompute at defined moments**, not continuously: at order placement, at courier assignment, at pickup. Each recomputation is a chance to correct. Each visible change is a small trust cost, so do not update the displayed number for every small improvement.

Round the output. A displayed "37 minutes" implies precision the model does not have. Ranges or 5-minute buckets are more honest and reduce complaints.

## 6. Keep it working

- **Retrain daily.** Traffic patterns, restaurant behavior, and courier supply all move.
- **Monitor by segment**, not in aggregate: by city, by hour, by restaurant. A new city with poor coverage is invisible in the global average.
- **Watch the feedback loop.** ETAs affect dispatch, dispatch affects the data, and the model learns from a world it shaped.
- **Weather and events are shocks.** Have a plan: a feature that captures them, and an alert when the error spikes.

## 7. Count the costs

Model inference is cheap. The cost is the streaming feature pipeline that keeps live traffic and restaurant load current, and the map data behind routing.

Levers: cache the route and its static features for repeated origin and destination pairs. Recompute the ETA at defined events, instead of on a timer.

## Common follow-ups

- **Why predict a distribution?** Because the displayed number is a business choice about how often you want to be late.
- **A new city with no history.** Transfer from similar cities, lean on the road network, and widen the buffer until data accumulates.
- **How do you handle an unusual event, like a storm?** Detect the error spike, widen the estimate, and be explicit with users rather than confidently wrong.
- **Why not one model end to end?** Because preparation and travel have unrelated drivers, and separate models can be debugged separately.

## Common mistakes

- Squared error loss, which treats early and late as equally bad.
- Reporting mean error and ignoring the tail.
- No real-time features, so the model is a historical average with extra steps.
- Showing a single precise minute value.

## Go deeper

The full worked solution, with the ETA capstone: [Grokking the AI System Design Interview](https://www.designgurus.io/course/grokking-the-ai-system-design-interview).
