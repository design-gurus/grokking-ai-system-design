# Design people you may know

> Suggest connections from a social graph, where the candidate set for one user can be tens of millions of people.

**Family:** Classic ML
**Also asked as:** design friend recommendations, design a connection suggestion system.

## 1. Frame the problem

Given a user, suggest people they know but are not connected to. Predict the probability that an invitation is sent and accepted.

**Both halves matter.** Optimizing invitations sent alone produces a system that suggests strangers with attractive profiles. Acceptance is the signal that the two people actually know each other.

The structure of the problem is a graph. The core idea is triangle closure: two people who share many connections probably know each other.

Scale to assume: 500 million members, an average of 300 connections each.

## 2. Define success

| Metric type | Metric | Target |
|-------------|--------|--------|
| Offline | Precision at 10 on held-out connections that later formed | Beat the current model |
| Online | Invitations sent, and the acceptance rate of those invitations | Both up |
| Guardrail | Invitations ignored or marked "I don't know this person", unsubscribe rate | Flat or better |

The guardrail is the whole ethics of this feature. A system tuned only for volume produces invitation spam, and the people receiving it leave.

## 3. Map the data

**The graph is the primary feature source.** For a candidate pair:

- Number of shared connections, which is the strongest single feature.
- Whether shared connections are themselves closely linked, which suggests a real community rather than a coincidence.
- Path length beyond two hops, which is much weaker.

**Off-graph signals matter and are sensitive:**

- Same employer, school, and years overlapping.
- Location.
- Imported contacts, if the user chose to share them.
- Profile view history, which is a strong signal and needs care in how it is used.

**Labels** are connections that formed, with the invitation and the acceptance recorded separately. Negatives are suggestions shown and ignored.

## 4. Pick the model

The hard part is not the model. It is generating candidates at all.

**Candidate generation** is friend-of-friend, and the arithmetic is brutal. A user with 1,000 connections who each have 1,000 connections has up to a million candidates, and popular accounts blow that up further.

Three controls, and you should name at least two:

1. **Cap the fan-out.** Ignore connections above a degree threshold, because a person with 30,000 connections tells you nothing about who knows whom.
2. **Prune by shared count.** Keep only candidates with at least k shared connections.
3. **Add non-graph sources.** Same company, same school, imported contacts. These cover users with a small graph, who are exactly the users the feature must help.

**Ranking** is then a gradient-boosted model over pair features. Simple, and it works. A graph neural network is a reasonable upgrade to mention, not the place to start.

## 5. The serving path

This is the classic case for precomputation, and saying so is the point of the question.

```
nightly:  [graph job] --> candidates per user --> [rank] --> store top N per user
serving:  [lookup] --> [filter: already connected, dismissed, blocked] --> [show]
```

The graph job is a distributed batch computation over hundreds of billions of pairs. It runs on a schedule, not per request.

**Filtering happens at serve time, not in the batch job.** Connections formed since the last run, dismissals, and blocks all have to be applied live, or users see suggestions for people they connected with yesterday. That is the most visible possible bug in this feature.

For high-value events, add a near-line path. When a user joins a company or imports contacts, recompute their suggestions within minutes.

## 6. Keep it working

- **Staleness has a budget.** Daily is usually acceptable, except right after a signup or an import.
- **Dismissals are permanent.** A suggestion the user rejected must never return.
- **New members are the hardest case.** They have no graph. Email contacts, employer, and school carry the whole feature until they connect to a few people.
- **Watch for a popularity loop.** Well-connected people get suggested more, which makes them more connected.

## 7. Count the costs

The nightly graph job dominates. Its cost is set by the fan-out cap, so that parameter is a cost decision as much as a quality one.

Storage is a ranked list per active member. Store it only for members who log in, not for all 500 million.

## Common follow-ups

- **How do you bound friend-of-friend?** Degree caps and a minimum shared-connection threshold. Give the arithmetic.
- **Why batch and not online?** The graph computation is far too expensive per request, and the answer changes slowly.
- **A brand new member.** Non-graph signals, and a near-line recompute after their first connections.
- **Privacy.** Some signals, like profile views, are sensitive. Say what you would and would not use as a visible reason for a suggestion.

## Common mistakes

- Proposing to compute friend-of-friend at request time.
- No fan-out cap, so popular accounts dominate every candidate set.
- Filtering already-connected people in the batch job instead of at serve time.
- Optimizing invitations sent without acceptance.

## Go deeper

- Written for a general system design round, architecture first: [Design people you may know](https://github.com/design-gurus/grokking-system-design/blob/main/questions/design-people-you-may-know.md).
- The full worked solution: [Grokking the AI System Design Interview](https://www.designgurus.io/course/grokking-the-ai-system-design-interview).
