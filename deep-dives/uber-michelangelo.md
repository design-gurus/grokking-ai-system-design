# Uber Michelangelo

> Uber's internal machine learning platform, and the reference shape for the answer to "how do many teams ship models without each building their own pipeline".

## What it is

Michelangelo is the platform Uber built to cover the full life of a model: managing data, training, evaluating, deploying, serving predictions, and monitoring them. It is worth knowing for two reasons. Most large companies converged on the same shape, and interviewers asking "design an ML platform" want that shape.

## The problem it solves

Before a platform, every team builds its own path from data to production. Each one writes its own feature code, its own training script, and its own serving service. Three problems follow:

- Feature definitions differ between training and serving, so models score worse in production than offline. That is training and serving skew.
- Nothing is shared, so the twentieth team repeats the work of the first.
- No one can answer basic questions later: which data trained this model, and which version is serving.

## Key design ideas

| Idea | How it works |
|------|--------------|
| A shared feature store | Features are defined once, stored as history for training and as latest values for low-latency serving. Teams reuse each other's features by name. |
| One workflow, six stages | Manage data, train, evaluate, deploy, predict, monitor. Every model follows it, which is what makes the platform usable by non-specialists. |
| Models as versioned artifacts | A trained model is packaged with its configuration, its feature list, and its evaluation report, so deployment and rollback are the same operation. |
| Both serving modes | The same model can run as a batch job that writes predictions to a table, or behind an online service that answers in milliseconds. |
| Evaluation stored with the model | Every training run records its metrics, so comparing two models is a query, not a search through old logs. |

## Trade-offs

- A platform is only worth it above a certain number of teams. For one team and one model, it is heavy.
- Standardizing the workflow limits unusual model types, and there is always a team whose model does not fit the standard path.
- A shared feature store creates shared failure: a broken backfill affects every model that reads that feature.
- Platform teams become a dependency for every product team, so the platform's own reliability becomes a company-level risk.

## What to say about it in an interview

Use it as the skeleton when asked to design an ML platform, and name the two things that justify it.

**The sentence to say.** "I would build the platform around a shared feature store and a versioned model registry. The feature store removes training and serving skew because both paths read one definition, and the registry makes deploy and rollback the same operation."

Expect the follow-up: how a team gets a new feature into production. The answer walks the pipeline. Define it once, backfill it over history, write it to both stores, then retrain.

## Go deeper

- Building block: [feature stores](../building-blocks/feature-stores.md)
- Course: [Grokking the AI System Design Interview](https://www.designgurus.io/course/grokking-the-ai-system-design-interview)
