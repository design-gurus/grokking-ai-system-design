# Batch vs online inference

> Compute the prediction before it is asked for, or when it is asked for. The choice sets your cost and your freshness.

## What it is

**Batch inference** runs the model on a schedule over many items at once and writes the results to a store. Serving is then a lookup.

**Online inference** runs the model inside the request, while the user waits.

## The problem it solves

Running a model per request is the obvious design and often the wrong one. A daily recommendation email for 10 million users does not need 10 million live model calls. Precomputing costs less, uses cheap off-peak capacity, and turns serving into a key lookup that cannot time out.

The opposite mistake is just as common. Precomputing a feed for every user wastes the work on the ones who do not log in. It also goes stale within minutes of the events that matter.

## How to choose

Ask two questions: **how stale can the answer be, and how many of the precomputed answers get used?**

| | Batch | Online |
|---|-------|--------|
| Freshness | Hours to a day old | Current at request time |
| Cost | Low, and it fills off-peak capacity | Pay per request, at peak |
| Latency | A lookup, single-digit milliseconds | Model latency is in the user's wait |
| Failure mode | Yesterday's answer is still served | The user sees an error or a timeout |
| Wasted work | Everything nobody looks at | None |
| Fits | Emails, daily digests, embeddings, scoring a whole catalog | Search, feeds, chat, fraud checks, anything using request context |

The tell is request context. If the prediction depends on what the user just typed, just clicked, or where they are, it cannot be precomputed.

## The near-line pattern

Most large systems land in the middle, and naming this is a senior signal.

```
batch:  compute candidates and heavy features for active users, hourly
online: score those candidates with fresh context, per request
```

The expensive, slow-moving part runs offline. The cheap, context-dependent part runs live. A feed does this: candidate generation is near-line, ranking is online. A [feature store](feature-stores.md) is what makes it work, because the precomputed values must be readable in a few milliseconds.

Batch only for the users who are likely to return. Precomputing for dormant accounts is the largest source of waste in these systems.

## Staleness has a budget

Say the number. "Recommendations may be up to one hour old" is a design decision with a cost attached, and it can be defended. "It refreshes periodically" is not.

Different parts of one system get different budgets. Item embeddings can be a day old. A user's session interests cannot be more than a few minutes old.

## Trade-offs

| Pro | Con |
|-----|-----|
| Batch is far cheaper per prediction | Batch answers are stale by design |
| Batch failures do not affect availability | A failed batch run silently serves old data, so it needs its own alert |
| Online reacts to context immediately | Model latency and cost enter every request |
| Online never computes an unused answer | Traffic spikes hit the model directly |

## Numbers to know

- Batch throughput is limited by the job window, not by per-item latency, so much larger models are affordable there.
- Online serving must fit a latency budget: tens of milliseconds for ranking, seconds for generation.
- A precomputed answer costs storage for every user, whether or not they return.
- Off-peak compute is cheaper on most clouds, and preemptible capacity is cheaper still.

## How to talk about it in an interview

**The sentence to say.** "Candidate generation runs hourly in batch for users active in the last week, because it is expensive and it changes slowly. Ranking runs online, because it depends on the current session. The staleness budget for candidates is one hour."

Expect these follow-ups:

- What happens to a user who has no precomputed row.
- How you detect a batch job that failed. Freshness of the output, not the exit code of the job.
- Where the cost actually is, and which half you would cut first.

## Go deeper

The full lesson on batch and online inference: [Grokking the AI System Design Interview](https://www.designgurus.io/course/grokking-the-ai-system-design-interview).
