# Drift detection and retraining

> Models get worse while nobody touches them, and the alert you need does not come from the model.

## What it is

Drift is the gap that opens between the world the model was trained on and the world it now serves. Nothing breaks. No error is raised. Predictions just get less useful.

## The problem it solves

A deployed model has no test suite that fails. Quality drops slowly, and the labels that would prove it arrive late or never. Without deliberate monitoring, the first signal is a business metric moving for reasons nobody can explain.

## The three kinds of drift

| Kind | What changes | How to detect it |
|------|--------------|------------------|
| Input drift | The distribution of a feature moves | Compare live feature distributions to the training set |
| Prediction drift | The distribution of the model's output moves | Track score histograms over time |
| Concept drift | The relationship between input and label changes | Only visible once labels arrive |

Concept drift is the dangerous one and the slowest to see. Fraud is the standard example: attackers change tactics specifically because your model learned the old ones.

## Detect the cheap things first

Input drift monitoring needs no labels, so it runs continuously and catches most real incidents.

Compare each feature's live distribution against a reference window, and alert on a large shift. The measure matters less than having one. Population stability index and a simple percentile comparison both work.

**The most common alert is not drift at all.** It is a broken feature pipeline: a job failed and the feature now returns zeros or nulls for everyone. The model accepts those values silently and scores badly. The same monitor catches both, which is a good argument for building it first.

## Then watch the model's output

Score distributions should be stable. A ranking model whose average predicted click rate doubles overnight did not get better. Something upstream changed.

Track the share of predictions near the decision threshold too. A classifier that becomes uncertain about everything is telling you the inputs no longer look like its training data.

## Retrain on a schedule or on a trigger

| Approach | Fits | Risk |
|----------|------|------|
| Fixed schedule, daily or weekly | Fast-moving data with quick labels, like feeds | Retrains when nothing changed, wasting compute |
| Triggered by a drift alert | Slower domains, expensive training runs | A bad trigger retrains on a broken pipeline |
| Continuous or online learning | Very fast-moving signals | Hard to debug, and a bad hour is learned immediately |

Whatever the trigger, the new model goes through the same gates as any other version. Automatic retraining that deploys automatically without an evaluation gate is how a broken pipeline becomes a broken model. See [model registry and rollout](model-registry-and-rollout.md).

## The feedback loop problem

A model trained on data its own predictions produced will drift toward its own preferences. A recommender that stops showing a category collects no interactions in that category, then concludes nobody wants it.

Two defenses: keep a share of exploration traffic, and hold out a control group that never sees the model, so you always have unbiased data.

## Numbers to know

- Feed and ads models: retrain daily or faster.
- Fraud and risk models: weekly, with the caveat that labels lag by 30 to 60 days.
- Language model prompts and retrieval corpora drift too. The corpus changes daily even when no model changes.
- Reference window for comparison: usually the last training set, or a fixed recent period.

## How to talk about it in an interview

**The sentence to say.** "I would monitor feature distributions against the training window, because that needs no labels and it catches both drift and a broken feature pipeline. Retraining runs on a schedule, and every retrained model still has to pass the evaluation gate before it can serve."

Expect these follow-ups:

- How you detect that quality dropped when labels take 60 days.
- What you would do if drift is detected but retraining does not help. The answer is that the features, not the model, are the problem.
- How you avoid retraining on data your own model shaped.

## Go deeper

- Evaluation layers: [evals](../evals/)
- Course: [Grokking the AI System Design Interview](https://www.designgurus.io/course/grokking-the-ai-system-design-interview)
