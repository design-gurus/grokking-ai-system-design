# Where training data comes from

> Labels are the scarcest thing in a machine learning system, and the interview question candidates answer worst.

## What it is

A label is the correct answer for one training example. Did the user click. Was this transaction fraud. Is this summary good. A model can only learn what the labels teach it, so the label supply decides what the system can do.

## The problem it solves

Interviewers ask "where does your training data come from" and get silence, or the answer "we would collect it". That answer says the candidate has never shipped a model.

The real question has three parts: what produces a label, how long it takes to arrive, and what it costs. Each one changes the design.

## The five sources

| Source | Example | Cost | Delay |
|--------|---------|------|-------|
| User behavior | A click, a purchase, a skip | Free, arrives with traffic | Seconds to minutes |
| A business outcome | A chargeback, a refund, a cancellation | Free, but rare | Days to months |
| Human annotation | A reviewer marks content as harmful | Dollars per item | Hours to days |
| Rules and heuristics | Flag every transaction over a threshold | Cheap | Instant, and noisy |
| Model generated | A larger model labels examples for a smaller one | Token cost | Minutes |

Most production systems use several at once. Behavior labels give volume. Human labels give a smaller, trusted set that measures the noisy ones.

## Label delay decides the design

This is the part to say out loud.

A click label arrives in seconds, so a feed model can retrain daily. A fraud chargeback label can arrive 60 days later, so today's fraud model is trained on a world two months old. A cancellation label for a subscription arrives after the billing cycle.

Long delay forces three things. A proxy label to train on in the meantime. A monitoring plan that does not depend on labels. A slower retraining cadence.

## The three traps

**Only what you showed gets labeled.** A ranking model sees interactions on the items it chose. Items it never showed have no data at all, so the training set slowly narrows to the model's own choices. A small share of exploration traffic keeps other items represented. See [online experiments](online-experiments.md).

**The positive class is tiny.** Fraud is under 1 percent of transactions. Harmful content is under 0.1 percent of posts. Accuracy is meaningless here, and so is a random sample. You oversample positives, or weight them, and you evaluate with precision and recall at a chosen threshold.

**Human labels disagree.** Two reviewers grading the same borderline post agree less often than teams expect. Measure the agreement rate first. If reviewers cannot agree, the guideline is unclear, and no model trained on those labels will do better than the confusion in them.

## Quality control for human labels

- Write the guideline with examples, including the borderline cases.
- Send a share of items to two or more reviewers and track the agreement rate.
- Mix in gold items with a known answer to score individual reviewers.
- Send the model's uncertain cases to review first. That is called active learning, and it buys far more accuracy per dollar than labeling random items.

## Numbers to know

- Human annotation runs from cents per item for simple tasks to dollars per item for expert review.
- A useful supervised baseline often needs a few thousand labeled examples, not millions.
- Fine-tuning a language model on a narrow task usually needs hundreds to a few thousand examples.
- An evaluation set of 500 to 2,000 well-chosen items measures more reliably than 100,000 noisy ones.

## How to talk about it in an interview

**The sentence to say.** "Positives come from user behavior, which arrives in seconds. The confirmed labels arrive 30 to 60 days later. I would train on the fast proxy, and use the slow label to check that the proxy still correlates with it."

Expect these follow-ups:

- What do you do before you have any labels. The answer is rules first, then swap in a model once the rules have produced data.
- How do you handle a rare positive class.
- How would you know your labels are wrong.

## Go deeper

The full lesson on the label supply chain: [Grokking the AI System Design Interview](https://www.designgurus.io/course/grokking-the-ai-system-design-interview).
