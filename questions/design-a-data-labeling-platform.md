# Design a data labeling platform

> A system whose output is training data, whose workers are people, and whose main quality problem is that two careful reviewers disagree.

**Family:** Platform
**Also asked as:** design an annotation tool, design a human review system, design Scale or Labelbox.

## 1. Frame the problem

Models need labeled examples. Somebody has to produce them, at volume, at a known quality level, for a known price.

The platform's job: take unlabeled items and a guideline, route them to the right people, measure whether the labels are trustworthy, and deliver a dataset.

The framing insight is that **label quality is bounded by guideline clarity**, not by worker effort. When two trained reviewers disagree 30 percent of the time, the fix is the guideline, not more reviewers.

Scale to assume: 500 reviewers across several vendors, 200,000 items labeled a day, dozens of concurrent projects.

## 2. Define success

| Metric type | Metric | Target |
|-------------|--------|--------|
| Quality | Agreement between reviewers on the same item | Stated per project, measured continuously |
| Quality | Accuracy against gold items with a known answer | Per reviewer, above a bar |
| Throughput | Items per reviewer hour, and end-to-end turnaround | Meets the project's deadline |
| Cost | Cost per labeled item, and cost per unit of model improvement | Falling |
| Coverage | Share of the model's uncertain cases that got labeled | High |

The last metric is the one that connects this platform to model quality. Labeling random items is far less valuable than labeling the items the model finds hard.

## 3. Map the data

- **Items.** The content to label, its source, and its priority.
- **Tasks.** One item assigned to one reviewer, with a state machine: assigned, submitted, in adjudication, final.
- **Labels.** The answer, the reviewer, the time taken, and the guideline version used.
- **Gold items.** Items with a known correct answer, mixed invisibly into the queue.
- **Guidelines, versioned.** A label is only meaningful together with the guideline version it was produced under.

Versioning the guideline is the step teams skip. When the policy changes, old labels describe the old rule, and mixing them into training silently teaches the model two contradictory things.

## 4. The core design choice

**Quality control by layered checks**, because you cannot inspect every label.

| Mechanism | What it catches | Cost |
|-----------|-----------------|------|
| Gold items in the stream | An individual reviewer drifting or guessing | Small, a few percent of volume |
| Overlap: the same item to 2 or 3 reviewers | Ambiguous items and unclear guidelines | 2 to 3 times on the overlapped share |
| Adjudication by a senior reviewer | Resolves disagreements, produces the final label | Expensive, applied to the disagreements only |
| Model as a reviewer | Flags labels that contradict the model strongly | Cheap, and it needs a human to confirm |

Overlap everything at the start of a project, when the guideline is unproven, then reduce it once agreement is stable. That schedule is the answer to "how do you keep quality high without tripling the cost".

**Active learning decides the queue.** The platform should pull the items the model is least certain about, plus a random sample for unbiased measurement. See [where training data comes from](../building-blocks/where-training-data-comes-from.md).

## 5. The serving path

```
[unlabeled pool] --> [sampler: uncertain items + random sample + gold]
                 --> [router: skills, language, vendor, availability]
                 --> [reviewer UI]
                 --> [submitted labels]
                 --> [quality checks: gold score, agreement]
                 --> [adjudication if needed]
                 --> [final dataset, versioned]
```

The reviewer interface is part of the design, not a detail. Seconds per item multiplied by 200,000 items a day is real money. Keyboard shortcuts, preloading the next item, and a fast decision layout are cost controls.

Routing must respect skill and language, and it must keep gold items indistinguishable from ordinary work.

## 6. Keep it working

- **Reviewers drift.** Scores from gold items are a per-person time series. Falling accuracy means retraining that person, not discarding them.
- **Guidelines get amended constantly.** Every amendment is a version, and it should trigger a small re-labeling of affected past items.
- **Watch for gaming.** A reviewer who always picks the majority class scores well on an imbalanced task. Track class distribution per reviewer.
- **Protect reviewers on harmful content projects.** Blurring, exposure limits, and rotation are requirements, not extras.

## 7. Count the costs

Cost per item multiplied by volume, plus the overlap multiplier, plus adjudication. Say the arithmetic out loud, because this platform's design is mostly a cost and quality trade.

Levers: reduce overlap as agreement stabilizes. Use a model to pre-label, so reviewers confirm rather than create. Route only uncertain items to expert reviewers.

Pre-labeling is a double-edged tool. It roughly doubles throughput and it biases reviewers toward accepting the model's answer, so keep a share of unassisted items to measure that effect.

## Common follow-ups

- **How do you know the labels are good?** Gold items per reviewer, and agreement between reviewers.
- **Two reviewers disagree. Now what?** Adjudication for the item, and a guideline fix if the pattern repeats.
- **How do you spend the labeling budget well?** Active learning on uncertain cases, plus a random sample to keep measurement honest.
- **What happens when the policy changes?** New guideline version, and re-label the affected slice.

## Common mistakes

- No gold items, so individual quality is unmeasured.
- One label per item on an ambiguous task.
- Labeling a random sample when the model needs its hard cases.
- Unversioned guidelines, so labels from different eras are mixed.

## Go deeper

- Related: [human in the loop](../building-blocks/human-in-the-loop.md)
- The full worked solution: [Grokking the AI System Design Interview](https://www.designgurus.io/course/grokking-the-ai-system-design-interview)
