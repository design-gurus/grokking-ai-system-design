# Feature stores

> One place that computes a feature once, then serves the same value to training and to production.

## What it is

A feature is a number the model reads as input. The count of a user's orders in the last 30 days is a feature. A feature store computes features, stores them, and serves them to two very different readers. A training job wants millions of rows of history. A live request wants one row in under 10 ms.

## The problem it solves

Without a feature store, the same feature is written twice. A data engineer writes it in SQL for the training set. A backend engineer writes it again in application code for the live request. The two drift apart, usually in a small way: one counts orders including cancellations, the other does not.

The model then sees a different world in production than it saw in training. This is called training and serving skew. It is the most common production failure in machine learning, and it is silent. There is no error, no alert, just a model that scores worse than the offline number promised.

The second problem is time. A training row must contain the feature value as it was at the moment the event happened, not as it is today. Joining today's value onto a label from last month leaks the future into the training set. The model looks brilliant offline and fails online. Feature stores solve this with a point-in-time join.

## How it works

```
sources --> transformations --> offline store (history, for training)
                            \-> online store  (latest value, for serving)
```

One definition of the feature is written once. Two paths carry it:

- The **offline store** holds the full history, usually in a warehouse or a data lake. Training jobs read it. It is optimized for large scans.
- The **online store** holds only the latest value per key, usually in a low-latency store like Redis or DynamoDB. The serving path reads it by user id or item id.

Features are grouped into sets that share a key and a refresh schedule. Slow features (a 30-day order count) are computed by a batch job every few hours. Fast features (clicks in the last five minutes) are computed by a stream job. Request-time features (the current search text, the device) are never stored at all, because they only exist during the request.

## When to use it

- More than a handful of models share features. Sharing is the main return.
- Features come from several sources and need to be joined the same way twice.
- You need point-in-time correctness, which is anything with a delayed label.

Skip it when there is one model, five features, and they all come from the request. A managed store is real infrastructure to run, and a small team often does better with a simple table and a scheduled job.

## Trade-offs

| Pro | Con |
|-----|-----|
| One definition removes training and serving skew | One more system to run and monitor |
| Point-in-time joins remove label leakage | Backfilling a new feature over history is expensive |
| Features are reusable across teams and models | Shared features create shared failure: one bad backfill affects every model that reads them |
| Online store gives predictable single-digit millisecond reads | Freshness has a cost, and the freshest features need a stream pipeline |

## Numbers to know

- Online feature read: 1 to 10 ms for a batch of keys.
- A ranking request usually fetches 50 to 500 feature values, in one or two round trips.
- Batch features refresh hourly or daily. Streaming features refresh in seconds.
- Storage: the offline store holds years of history, so it is usually the larger cost.

## How to talk about it in an interview

The signal is that you noticed the two readers. Say that the training path and the serving path must use the same feature definition, and name the failure that happens when they do not.

**The sentence to say.** "I would define these features once in a feature store. Training reads the offline history with a point-in-time join, so no future data leaks in. Serving reads the latest values from the online store in under 10 ms."

Expect these follow-ups:

- Which features are batch and which are streaming, and why.
- What happens when the online store is missing a key for a new user. The answer is a default value, and the model must have seen that default during training.
- How you would add a new feature without retraining everything.

## Go deeper

The full lesson, including the encoding recipes for turning raw signals into model inputs: [Grokking the AI System Design Interview](https://www.designgurus.io/course/grokking-the-ai-system-design-interview).
