# Design a recommendation system

> Pick what to show a user from a catalog of millions, when they did not ask for anything specific.

**Family:** Classic ML
**Also asked as:** design Netflix recommendations, design "customers also bought", design a discovery page.

## 1. Frame the problem

There is no query. The system has to guess intent from history, which is what separates this from search.

The task: given a user, a context, and a catalog item, predict the probability of the action that matters. Say which action out loud, because it changes everything downstream. A watch, a purchase, a save, and a click are four different systems.

Scale to assume: 50 million users, a catalog of 10 million items, 20 million recommendation requests a day.

## 2. Define success

| Metric type | Metric | Target |
|-------------|--------|--------|
| Offline | Recall at 100 for retrieval, AUC for ranking | Beat the current model by a stated margin |
| Online | Conversion rate, revenue per session, or completed views | Up 1 to 3 percent in an A/B test |
| Guardrail | p99 latency, catalog coverage, share of impressions on the top 1 percent of items | Latency under 300 ms, coverage flat or better |

Catalog coverage is the guardrail candidates forget. A model that maximizes clicks converges on a few thousand popular items. Short-term metrics improve, the long tail stops selling, and the catalog stops being worth carrying.

## 3. Map the data

**Labels are implicit.** Views, clicks, add-to-cart, purchase, and completion. Explicit ratings barely exist, and the users who leave them are not a random sample.

Three problems to name:

- **Popularity bias.** Popular items get shown more, so they collect more positives, so the model shows them more. This is a loop, not a preference.
- **Position bias.** The top slot gets clicked because it is the top slot. Include position as a training feature and set it to a constant at serving time.
- **Negatives are not obvious.** An item shown and ignored is a weak negative. An item never shown is not a negative at all. Sampling strategy matters more than model choice here.

**Features** in three groups: user history and demographics, item content and statistics, and interaction features like affinity to this category. See [feature stores](../building-blocks/feature-stores.md).

## 4. Pick the model

Ladder, in order:

1. **Popularity by segment.** The baseline that must be beaten, and it is stronger than people expect.
2. **Collaborative filtering.** Learn user and item vectors from the interaction matrix. Works well and cannot handle a new item at all.
3. **[Two-tower retrieval](../building-blocks/two-tower-retrieval.md) plus a ranker.** The production shape: cheap retrieval over the catalog, then a gradient-boosted or neural ranker over a few hundred candidates.

Content features are the answer to cold start. An item with a title, a category, and images has a vector on day one, before anyone interacts with it.

## 5. The serving path

Budget: 300 ms.

```
[request] --> [candidate sources, in parallel]
                two-tower retrieval | recently viewed | popular in segment | exploration
          --> [merge and dedupe, ~500 items]
          --> [ranker, 60 ms]
          --> [business rules: in stock, region, already owned]
          --> [diversity pass] --> [response]
```

Availability rules go after the model, never inside it. A model that learned to avoid out-of-stock items has learned last week's inventory.

Precompute or score live? For a homepage that changes slowly, precompute per active user every few hours and score live only when the session provides new signal. See [batch vs online inference](../building-blocks/batch-vs-online-inference.md).

## 6. Keep it working

- **Retrain daily.** Item popularity moves, seasons change, and the catalog turns over.
- **Reserve exploration traffic.** Without it the model never learns about items it does not already favor.
- **Watch coverage and long-tail sales**, not only the primary metric.
- **Cold start for users.** Popular by segment, then adapt within the first session. Say how quickly the first few clicks change the results.
- **Filter what should never appear.** Already purchased, already dismissed, out of region. Keep this list in serving, where it is auditable.

## 7. Count the costs

20 million requests a day at 500 candidates each is 10 billion scores a day. That number is the argument for a small ranker and a hard cap on candidates.

Levers: shrink the candidate set for low-activity users, cache the ranked list for a few minutes, and precompute for users who visit daily.

## Common follow-ups

- **A brand new item.** Content features plus a slice of exploration traffic, so it collects interactions before it can be ranked properly.
- **A brand new user.** Segment popularity, then in-session adaptation. Say what signal you use in the first 30 seconds.
- **Why not just show popular items?** It is the baseline, and it beats a bad personalized model. Personalization has to earn its complexity in an experiment.
- **How do you stop it recommending the same thing forever?** A diversity rule, a repeat penalty, and dismissal signals.

## Common mistakes

- One model that both retrieves and ranks.
- Treating unseen items as negatives.
- Ignoring popularity bias, then reporting a strong offline AUC.
- Putting inventory and region rules inside the model.

## Go deeper

- Written for a general system design round, architecture first: [Design a recommendation system](https://github.com/design-gurus/grokking-system-design/blob/main/questions/design-recommendation-system.md).
- The full worked solution: [Grokking the AI System Design Interview](https://www.designgurus.io/course/grokking-the-ai-system-design-interview).
