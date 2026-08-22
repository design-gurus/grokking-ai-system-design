# Design fraud detection

> Score a payment in 100 ms, when the label proving you were right arrives 60 days later and an adversary is adapting to you.

**Family:** Classic ML
**Also asked as:** design payment fraud detection, design account takeover detection, design a risk engine.

## 1. Frame the problem

Given a transaction, decide: allow, challenge, or block. Three outcomes, not two, and the middle one is what makes the system usable.

Three properties make this the hardest classic ML question:

1. **Labels arrive late.** A chargeback can take 30 to 90 days.
2. **Positives are rare.** Well under 1 percent of transactions.
3. **The data fights back.** Attackers change behavior because your model learned the old behavior.

Scale to assume: 5,000 transactions per second at peak, a 100 ms budget inside the payment path.

## 2. Define success

Accuracy is meaningless at a 0.3 percent positive rate. Use a cost matrix instead, and say the numbers.

| Outcome | Cost |
|---------|------|
| Fraud allowed | The transaction value, plus a chargeback fee |
| Good transaction blocked | Lost margin, plus a customer who may not return |
| Challenge shown | Some share of good customers abandon |

| Metric type | Metric | Target |
|-------------|--------|--------|
| Offline | Precision and recall at the operating threshold, and the cost-weighted total | Stated per launch |
| Online | Fraud loss rate in basis points, approval rate for good customers | Loss down, approvals flat |
| Guardrail | p99 latency, false positive rate on established customers, manual review volume | Latency under 100 ms |

The false positive rate on good customers is the guardrail that keeps the business from turning off the model. Blocking legitimate spending is a visible, complained-about failure. Fraud is a quiet one.

## 3. Map the data

**Labels come from three places, at three speeds:**

| Source | Delay | Quality |
|--------|-------|---------|
| Customer reports a disputed charge | Days | Good |
| Chargeback from the network | 30 to 90 days | Best, and legally definitive |
| Manual review decision | Hours | Good, and available now |

The delay is why manual review is not just a safety net. It is the fast label supply, and it is what you train on while chargebacks catch up. See [human in the loop](../building-blocks/human-in-the-loop.md).

**Features that carry the signal:**

- **Velocity.** Transactions per card, device, or IP in the last minute, hour, and day. Streaming counters.
- **Entity linkage.** How many accounts share this device, how many cards share this address. Graph features find rings that per-transaction features cannot.
- **Deviation from this customer's own history.** Amount, merchant category, hour, country.
- **Third-party signals.** Issuer response codes, device fingerprint reputation.

## 4. Pick the model

Rules and a model, together. This is not a compromise, it is the correct design.

- **Rules** handle known patterns and give you an instant response to a new attack. A model needs a retrain, and by then the attack has run for a week.
- **Gradient-boosted trees** on a few hundred features handle the general case. They train quickly, and their feature importances are explainable, which matters for a regulated decision.
- **Anomaly detection** catches patterns that have no labels yet, at the cost of many false positives. Route its output to review rather than to a block.

Handle imbalance with class weights or by downsampling negatives, and correct the resulting scores if you need calibrated probabilities.

## 5. The serving path

Budget: 100 ms, inside the payment authorization path.

```
[transaction] --> [feature fetch: velocity counters, entity graph, history]  30 ms
              --> [rules engine]  1 ms
              --> [model]  20 ms
              --> [decision policy: allow / challenge / block]
              --> [async: write to review queue if uncertain]
```

**Decide the failure policy explicitly.** If the model is unavailable, do you allow everything or block everything? Allowing is the usual answer, with rules still enforced and an alert raised. Blocking all payments is a worse outage than a few hours of higher fraud. Say it out loud, because it is a business decision the interviewer wants to hear you make.

## 6. Keep it working

- **Champion and challenger.** Run the new model in shadow on live traffic. Compare decisions before you switch.
- **Retrain on a schedule**, accepting that the newest label window is incomplete.
- **Watch the score distribution daily.** A sudden shift is either an attack or a broken feature pipeline. See [drift detection](../building-blocks/drift-detection-and-retraining.md).
- **Keep a reserve of unblocked traffic.** If you block every suspicious transaction, you never learn whether those were actually fraud. A tiny holdout, or a challenge instead of a block, keeps the label supply honest.

## 7. Count the costs

Model serving is cheap here. The real costs are the streaming feature infrastructure, the manual review team, and the fraud losses themselves.

Do the review math out loud: transactions routed to review per day, divided by cases per reviewer per hour, gives headcount. That number decides where you set the threshold.

## Common follow-ups

- **How do you evaluate with 60-day labels?** Manual review labels now, chargebacks as the delayed check, and a stable evaluation window from the past that is fully labeled.
- **A new attack pattern appears today.** A rule within hours, retrain within days, and a review queue in between.
- **Why not just block everything suspicious?** The cost matrix, and the customers you lose.
- **Explainability.** A blocked customer may be entitled to a reason, which favors models whose decisions can be described.

## Common mistakes

- Reporting accuracy on an imbalanced problem.
- Ignoring the label delay, and assuming yesterday's data is labeled.
- No rules layer, so a new attack runs until the next retrain.
- Forgetting that blocking prevents you from learning.

## Go deeper

The full worked solution: [Grokking the AI System Design Interview](https://www.designgurus.io/course/grokking-the-ai-system-design-interview).
