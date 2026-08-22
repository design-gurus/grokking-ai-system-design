# Human in the loop

> Sending the uncertain and the expensive cases to a person, which is a design choice with a staffing number attached.

## What it is

The system does not decide everything. Cases above a risk threshold, or below a confidence threshold, go to a queue where a person decides. The person's decision becomes the outcome, and also a label.

## The problem it solves

Full automation on a task the model gets right 92 percent of the time means 8 percent wrong outcomes, at volume, with no one checking. For a recommendation that is fine. For a refund, an account suspension, or a medical triage message, it is not.

Routing the hard cases to people is the difference between a system that can ship and one that stays a prototype.

## The three placements

| Placement | The person acts | Fits |
|-----------|-----------------|------|
| Before the action | Approves or rejects the model's proposal | Irreversible or expensive actions |
| After the action | Reviews a sample after the fact | High volume, reversible, low individual cost |
| Instead of the action | Takes over the whole case | The model declined or hit a limit |

Before-the-action review is the expensive one, so it is reserved by threshold. A refund under a limit goes through automatically. Above it, the agent prepares the action and a person approves. Raising that limit as evidence accumulates is a good thing to offer in an interview.

## Deciding what to route

Three signals, and using more than one is better than tuning any of them:

- **Model confidence.** Useful, and it must be calibrated first. An uncalibrated 0.9 means nothing. Compare predicted probabilities against observed rates before trusting them.
- **Value at risk.** Route by amount, account tier, or reversibility, regardless of confidence.
- **Disagreement.** Two models, or a model and a rule, that disagree is a strong signal for review.

A share of random traffic should also go to review. Routing only the uncertain cases means you never measure quality on the confident ones, which is exactly where a silent failure hides.

## The queue is a system

The review queue has all the properties of any queue, and forgetting that is a common interview miss.

- Arrival rate against service rate. If reviewers cannot keep up, the backlog grows without limit.
- Priority, so urgent cases do not sit behind routine ones.
- A timeout policy. When nobody reviews within the window, what happens: auto-approve, auto-reject, or page someone. Choose deliberately, because the default is a case that waits forever.
- Escalation latency as a user-facing metric. Time to human decision belongs on the dashboard.

## Reviews are training data

Every human decision is a label on a case the model found hard, which is the most valuable kind. Feed those into the training set and into the evaluation set.

This is active learning, and it is why human review gets cheaper over time. The cases people fix become the cases the model stops getting wrong. See [where training data comes from](where-training-data-comes-from.md).

## Trade-offs

| Pro | Con |
|-----|-----|
| Hard cases get judgment, not a guess | People cost money and do not scale with traffic |
| Irreversible actions become safe to automate | Review adds latency, sometimes hours |
| Reviewed cases are high-value training labels | A backlog turns into a silent outage |
| Thresholds can loosen as evidence accumulates | Reviewers rubber-stamp when volume is too high |

## Numbers to know

- Do the staffing math out loud: cases routed per day, divided by cases a reviewer handles per hour, gives the headcount.
- A review rate above roughly 20 to 30 percent usually means the automation is not carrying its weight yet.
- Rubber-stamping starts when volume per reviewer is too high. Sample their agreement with a second reviewer to detect it.
- Timeout windows are a product decision: minutes for a live chat, hours for a refund.

## How to talk about it in an interview

**The sentence to say.** "Refunds under the threshold are automatic. Above it, the agent prepares the action and a person approves, with a timeout policy so nothing waits forever. Reviewed cases go straight into the training and evaluation sets, and I would raise the threshold as the confirmed error rate drops."

Expect these follow-ups:

- How many reviewers this needs. Do the arithmetic.
- What happens when the queue backs up.
- How you stop reviewers approving everything without reading it.

## Go deeper

- Related: [guardrails and safety filters](guardrails-and-safety-filters.md), [design a customer support agent](../questions/design-a-customer-support-agent.md)
- Course: [Grokking the AI System Design Interview](https://www.designgurus.io/course/grokking-the-ai-system-design-interview)
