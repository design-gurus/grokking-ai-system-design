# Online experiments

> The only step that decides whether a model launches, and the reason offline scores are a filter rather than a verdict.

## What it is

An online experiment splits live users into groups. One group gets the current system, one gets the change. You compare the metrics that matter to the business, not the ones that are convenient to compute.

## The problem it solves

Offline scores measure the model against logged history. Logged history was produced by the old model, it contains no reaction to the new one, and it cannot show second-order effects.

A model can win offline and lose online for ordinary reasons. It is slower, so users see fewer results. It promotes items that get clicks and lose trust. It helps new users and hurts loyal ones. Only live traffic shows this.

## How it works

```
users --> hash(user id) --> 95% control (current model)
                        --> 5%  treatment (new model)
```

Assignment is by a stable hash of the user id, so the same user stays in the same group for the whole experiment. Assigning per request instead of per user is a classic error: the user sees an inconsistent product, and the measurement is polluted.

Three metric groups, decided before the experiment starts:

| Group | Role | Example |
|-------|------|---------|
| Primary | The number that decides launch | Session time, conversion, resolved tickets |
| Secondary | Explains the primary movement | Click rate, items viewed |
| Guardrail | Blocks launch regardless of the primary | p99 latency, reported content, cost per request |

Naming the primary metric in advance stops the most common bad practice, which is running the test and then picking whichever metric moved.

## The variants worth knowing

| Method | Use it when | Why |
|--------|-------------|-----|
| A/B test | The default | Simple, unbiased, well understood |
| Interleaving | Comparing two rankers | Mixes both lists for the same user, so it needs far less traffic |
| Switchback | Marketplaces and logistics | Splits by time window, because users and drivers affect each other |
| Bandit | Many variants, one cheap metric | Shifts traffic toward winners while the test runs |

Interleaving and switchback exist because plain A/B breaks in specific situations. Interference between groups is the reason. If treatment users change the experience of control users, for example by taking the same delivery driver, a user-level split measures nothing.

## What goes wrong

**Novelty effect.** Anything new gets clicks in week one. Run long enough to see the effect settle.

**Underpowered tests.** A small effect on a small share of traffic cannot be detected. Estimate the sample size first. If the traffic does not exist, say so. The change cannot be measured, and shipping it is a judgment call rather than a result.

**Peeking.** Checking daily and stopping the moment the number looks good turns a 5 percent error rate into a much larger one. Fix the duration up front.

**Segment blindness.** An average that does not move can hide a large gain for new users and a large loss for loyal ones. Break results down by the segments you already care about.

## Numbers to know

- Typical run: one to four weeks. Long enough for novelty to fade and for weekly cycles to repeat.
- Ramp pattern: 1 percent, then 5, then 50, then full.
- Interleaving can detect a ranking difference with roughly an order of magnitude less traffic than an A/B test.
- Guardrails are checked continuously, not at the end. A latency regression should stop the test on the first day.

## How to talk about it in an interview

**The sentence to say.** "Offline scores decide which candidate is worth an experiment. Launch is decided by an A/B test on session time, with p99 latency and reported content as guardrails, run for two weeks so novelty settles."

Expect these follow-ups:

- How you would compare two rankers with limited traffic. The answer is interleaving.
- Why a marketplace cannot use a user-level split.
- What you do when the primary metric is flat and a secondary one improved.

## Go deeper

- The three evaluation layers: [evals](../evals/)
- Course: [Grokking the AI System Design Interview](https://www.designgurus.io/course/grokking-the-ai-system-design-interview)
