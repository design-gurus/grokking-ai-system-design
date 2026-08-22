# Design a feature store

> One definition of a feature, served to a training job that wants years of history and to a live request that wants one row in 5 ms.

**Family:** Platform
**Also asked as:** design an ML feature platform, design Feast or Michelangelo's feature layer.

## 1. Frame the problem

Teams keep writing the same feature twice: once in SQL for training, once in application code for serving. The two drift, and the model sees a different world in production than it saw in training. That is training and serving skew, and it is the failure this platform exists to prevent.

The platform's job: let a team define a feature once, then serve correct values to both readers.

Scale to assume: 40 teams, 5,000 feature definitions, 200,000 online reads per second, years of history offline.

## 2. Define success

| Metric type | Metric | Target |
|-------------|--------|--------|
| Correctness | Skew: online values compared against offline values for the same key and time | Effectively zero |
| Service | p99 online read latency for a batch of features | Under 10 ms |
| Freshness | Age of the newest value, per feature group | Meets each group's declared target |
| Adoption | Share of production models reading from the store | Rising |
| Productivity | Time from defining a feature to using it in training | Hours, not weeks |

Skew is the metric that justifies the platform. Measure it by logging the feature values used at serving time and comparing them against what the offline store would have produced.

## 3. Map the data

Two stores, one definition.

| | Offline store | Online store |
|---|---------------|--------------|
| Holds | Full history, all versions | Latest value per key |
| Shape | Columnar, in a warehouse or lake | Key-value, in memory or a fast database |
| Read pattern | Large scans over time ranges | Point lookups by entity id |
| Reader | Training and backfill jobs | The serving path |

**Log the values you served.** A row that records what serving actually read, keyed by request, is what makes skew measurable. It also lets you rebuild a training set that matches reality.

## 4. The core design choice

**Point-in-time correctness.** This is the hardest requirement and the thing an interviewer is listening for.

A training row for an event last month must contain the feature values as they were at that moment, not today's values. Joining today's values onto an old label leaks the future into training, and the model looks excellent offline and fails online.

So the offline store keeps every value with its valid-from timestamp. The training join asks for the latest value as of the event time, not the current value.

**One definition, two materializations.** A feature is declared once: its name, its entity key, its transformation, its source, and its freshness target. The platform then generates both paths. One job writes history to the offline store. Another keeps the online store current.

**Freshness tiers**, declared per feature group:

| Tier | Refresh | Example |
|------|---------|---------|
| Batch | Hourly or daily | 30-day order count |
| Streaming | Seconds | Clicks in the last five minutes |
| Request time | Not stored at all | The current search text, the device |

Request-time features never enter the store. Saying that shows you know what the store is for.

## 5. The serving path

```
training:  [definitions] --> [batch job] --> [offline store] --> point-in-time join --> [training set]
serving:   [request] --> [online store: multi-get by entity ids] --> [model]
                     --> [log the served values]
```

Online reads are batched. One request may need 200 feature values across three entity types, so it must be two or three multi-key reads, never 200 round trips.

**Missing values need a defined default.** A brand new user has no history. The default served in production must be the same default used in training, or you have created skew deliberately.

## 6. Keep it working

- **Backfill is the expensive operation.** Adding a feature means computing it over history so old training rows can use it. Plan for it, meter it, and expect it to dominate cost.
- **A shared feature is a shared dependency.** A bad backfill affects every model reading that feature. Version definitions, and never change one in place.
- **Monitor freshness per group**, and alert when a value is older than its declared target. A stalled job that serves stale values raises no error anywhere else.
- **Deprecate.** Of 5,000 definitions, most are unused. Track readers per feature and retire the rest, or the platform collapses under its own catalog.

## 7. Count the costs

The offline store holds years of history for thousands of features, so storage is the largest line. The online store is sized by hot keys and is usually memory-bound.

Levers: keep only the entities that are actually served online. Set retention per feature group. Compute expensive features only when a model declares it reads them.

## Common follow-ups

- **What is a point-in-time join and why does it matter?** The core answer for this question.
- **What happens when a feature is missing at serving time?** A declared default, matched in training.
- **How do you know serving and training agree?** Log served values and compare.
- **Do streaming features need a different store?** The same online store, a different writer.

## Common mistakes

- One store, and no story for point-in-time correctness.
- Fetching features one at a time in the request path.
- Allowing feature definitions to be edited in place.
- No freshness monitoring, so a stalled pipeline is invisible.

## Go deeper

- Building block: [feature stores](../building-blocks/feature-stores.md)
- Platform shape: [Uber Michelangelo](../deep-dives/uber-michelangelo.md)
- The full worked solution: [Grokking the AI System Design Interview](https://www.designgurus.io/course/grokking-the-ai-system-design-interview)
