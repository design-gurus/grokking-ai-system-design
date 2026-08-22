# Design harmful content detection

> Find policy-violating content across a billion posts a day, where both mistakes are expensive and the definition of the problem is written by people, not by data.

**Family:** Classic ML
**Also asked as:** design content moderation, design trust and safety, design spam and abuse detection.

## 1. Frame the problem

Classify content against a policy, then take an action. The action is not a single choice, and saying so early is what separates a real answer from a classifier description.

**The action ladder**, from cheapest to most severe. Leave it. Reduce its distribution. Add a warning or an age gate. Send it to human review. Remove it. Act on the account.

Confidence decides where you land on that ladder. High confidence removes. Medium confidence demotes and queues for review. That is the design.

Scale to assume: 500 million posts a day, text plus images plus video, dozens of languages, and a policy document that changes monthly.

## 2. Define success

| Metric type | Metric | Target |
|-------------|--------|--------|
| Offline | Precision and recall per policy area | Stated per policy, not overall |
| Online | Prevalence: the share of views that are of violating content | The number leadership tracks |
| Online | Time to action on a violating post | Minutes for severe categories |
| Guardrail | Appeals overturned, legitimate content removed | Overturn rate under a stated limit |

**Prevalence, not volume.** A million removals sounds impressive and says nothing. Prevalence measures what users are actually exposed to, and it is measured by sampling and human-labeling ordinary traffic, not by counting enforcement actions.

Metrics must be per policy area. Precision that is acceptable for spam is not acceptable for a category where removal has legal weight.

## 3. Map the data

**Labels come from human reviewers, working from a written guideline.** That guideline is the real specification of the system.

Two consequences to name:

- **Reviewer agreement caps model quality.** If two trained reviewers agree only 70 percent of the time on a category, no model will do better than the confusion in the labels. Measure agreement first, and treat a low number as a guideline problem.
- **Policy changes invalidate labels.** When the policy shifts, old labels describe the old rule. You need a re-labeling plan, and a way to know which model version was trained on which policy version.

**Other signals:** user reports, which are high volume and heavily gamed, and account-level history, which is often stronger than the content itself.

The data is also multilingual and multimodal, and both dimensions are places where coverage silently ends. Say which languages are covered and what happens in the ones that are not.

## 4. Pick the model

**A risk funnel**, because scoring everything with an expensive model is impossible at this volume.

```
all content --> [cheap filters: hashes of known bad, rules, keyword lists]
            --> [small classifier on everything]      ~1% flagged
            --> [expensive multimodal model on the flagged slice]
            --> [human review on the uncertain slice]
```

Exact-match hashing catches re-uploads of known violating media at almost no cost, and it handles a large share of severe categories. Do that first.

Per-policy classifiers beat one model with many labels, because thresholds, review policies, and precision targets differ per category.

## 5. The serving path

Two paths, and they have different budgets.

**At upload, asynchronous.** Content is queued, scored, and actioned within seconds to minutes. Severe categories jump the queue.

**At view, synchronous.** Look up the stored decision, which must be a fast cache read. Never run a classifier in the request path of a page view.

```
[upload] --> [queue] --> [funnel] --> [decision store]
[view]   --> [decision store lookup] --> [show, demote, or block]
```

Re-scoring matters: when a model or policy changes, decisions for existing content have to be recomputed in the background, oldest and highest-reach content first.

## 6. Keep it working

- **Adversaries adapt.** Text is misspelled, images are cropped and re-encoded, words are replaced with lookalikes. Expect measurable decay in weeks, not months.
- **Sample and label ordinary traffic continuously.** It is the only way to measure prevalence and to catch the categories your model never flags.
- **Appeals are a labeled dataset.** An overturned removal is a precision failure with a free label attached.
- **New harm types appear.** The response is rules and human review first, then a model once labels exist.

## 7. Count the costs

Human review is the dominant cost, and it is a headcount number: items routed to review per day, divided by items per reviewer per hour.

That makes the funnel's job explicit. Every step exists to shrink what reaches a person, without lowering prevalence performance.

Reviewer welfare is a real design consideration for severe categories: blurring, limits on exposure, and rotation.

## Common follow-ups

- **What do you do when the model is unsure?** Demote and review. The middle of the action ladder exists exactly for this.
- **How do you handle a language with no labeled data?** Multilingual models plus translation as a stopgap, and be explicit that quality is lower there.
- **How fast can you respond to a new attack?** Hashes and rules in hours, a model in days.
- **How do you measure what you never catch?** Sampled human labeling of ordinary traffic. Enforcement counts cannot answer this.

## Common mistakes

- One binary classifier and one threshold for every policy.
- Measuring removals instead of prevalence.
- Running an expensive model on everything.
- No plan for appeals, re-scoring, or policy changes.

## Go deeper

The full worked solution: [Grokking the AI System Design Interview](https://www.designgurus.io/course/grokking-the-ai-system-design-interview).
