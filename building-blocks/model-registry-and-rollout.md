# Model registry and rollout

> Shipping a model the same way you ship code: versioned, staged, and reversible in one step.

## What it is

A model registry holds every trained model as a versioned artifact. Each version carries what is needed to serve it and to trust it. Rollout is how a version moves from registered to serving live traffic.

## The problem it solves

Without a registry, the model in production is a file someone copied to a server. Three questions then have no answer: which data trained it, which version is live, and how do you go back.

Going back matters most. Model quality problems are found in production, hours or days after a change. If rollback needs a retrain, the outage lasts as long as the retrain.

## What a registered version contains

| Part | Why it is needed |
|------|------------------|
| The weights | The model itself |
| The feature list and their versions | Serving must send exactly the inputs training used |
| Preprocessing code or config | A mismatch here is silent and produces bad scores |
| Training data reference | Reproducing or debugging the run later |
| Evaluation report | Comparing candidates without rerunning them |
| Metadata: author, time, parent version | Audit, and finding what changed |

The model and its preprocessing must be versioned together. Pinning one without the other is the most common cause of a model that scores well offline and badly online.

## The rollout ladder

Move down this ladder as confidence grows. Each step catches a different failure.

1. **Shadow.** The new model scores live traffic, and its output is discarded. This catches crashes, latency problems, and feature pipeline errors. It cannot measure quality, because nobody sees the results.
2. **Canary.** A small share of real traffic, usually 1 to 5 percent. Watch errors, latency, and guardrail metrics. This catches problems the offline set never contained.
3. **Experiment.** A proper A/B test that measures the online metric. This is the only step that decides a launch. See [online experiments](online-experiments.md).
4. **Full rollout**, with the previous version still registered and one config change away.

## Rollback is a config change

The serving layer reads which version to use from configuration, not from a build. Rolling back is then a config edit that takes effect in seconds. It needs no deploy, no rebuild, and not the person who trained the model.

Say this out loud in an interview. It is a short sentence and it separates people who have operated a model from people who have trained one.

## When to use it

Any system with more than one model or more than one person changing it. For a single model owned by one engineer, a versioned artifact in object storage with a metadata file covers most of the value.

## Trade-offs

| Pro | Con |
|-----|-----|
| Deploy and rollback are the same operation | One more system in the path to production |
| Model comparison is a query, not a spreadsheet | Registry metadata rots unless writing it is automatic |
| Reproducibility for audit and debugging | Storing every version and its data costs real money |
| Staged rollout catches what offline sets miss | Each stage adds days before a change reaches users |

## Numbers to know

- Ranking models often ship daily or several times a week.
- Shadow runs for hours. Canary runs for a day or two. An experiment runs one to four weeks, long enough to see retention effects.
- Keep the last several versions immediately loadable. Older ones can move to cold storage.

## How to talk about it in an interview

**The sentence to say.** "Every model is a registered version with its features and preprocessing pinned to it. Serving reads the active version from config, so rollback is a config change and takes seconds. New versions go shadow, then canary, then an A/B test."

Expect these follow-ups:

- How you notice that a new model is worse, and how fast.
- What you do when the new model is better on average and worse for one segment.
- How the feature pipeline is versioned alongside the model. See [feature stores](feature-stores.md).

## Go deeper

- Platform shape: [Uber Michelangelo](../deep-dives/uber-michelangelo.md)
- Course: [Grokking the AI System Design Interview](https://www.designgurus.io/course/grokking-the-ai-system-design-interview)
