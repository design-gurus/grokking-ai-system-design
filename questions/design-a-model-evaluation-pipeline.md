# Design a model evaluation pipeline

> The test suite for a system that has no deterministic output, run on every prompt change and every model version.

**Family:** Platform
**Also asked as:** design an eval platform, design regression testing for LLMs.

## 1. Frame the problem

Teams change prompts, models, and retrieval settings several times a week. Without an automated way to score those changes, every release is a guess, and quality regressions are found by users.

The platform's job: given a candidate system and a dataset, produce comparable scores. Fast enough to gate a pull request, and cheap enough to run often.

Scale to assume: 30 teams, 200 evaluation datasets, thousands of runs a week, datasets from 100 to 50,000 examples.

## 2. Define success

| Metric type | Metric | Target |
|-------------|--------|--------|
| Trust | Agreement between the pipeline's judgment and human review | Measured per judge, above a stated bar |
| Stability | Variance when the same candidate is scored twice | Small enough that real changes are visible |
| Speed | Time for a gating run in continuous integration | Under 15 minutes |
| Cost | Spend per run, and per team per month | Predictable and capped |
| Adoption | Share of production changes that ran evaluation first | Near total |

**Stability is the metric people skip.** If rerunning the same candidate moves the score by 3 points, a 2-point improvement means nothing. Fix variance before trusting any comparison: pin sampling temperature to zero where possible, fix the dataset order, and report a confidence interval.

## 3. Map the data

- **Datasets, versioned.** Inputs, expected outputs where they exist, and metadata. Never edited in place, because changing the dataset changes the score and invalidates history.
- **Runs.** Candidate identifier, dataset version, judge version, per-example results, and aggregate scores.
- **Per-example outputs.** Kept, not just aggregates. Debugging a regression means looking at the examples that changed.
- **Human labels.** From review sampling, used to calibrate the judges.

Datasets come from three places: hand-written cases, real production traffic that was reviewed and labeled, and cases added after each incident. The third source is the most valuable, and it grows the suite in exactly the right direction.

## 4. The core design choice

**A scorer registry, not one scoring method.** Different tasks need different checks, and the pipeline should treat them uniformly.

| Scorer | Cost | Fits |
|--------|------|------|
| Exact or schema match | Free | Structured output, classification |
| Programmatic check | Free | Does the code run, is the JSON valid, is the citation real |
| Retrieval metrics | Cheap | Recall at k for the retrieval stage |
| Model as judge | Expensive | Open-ended text quality |
| Human review | Very expensive | Calibrating the judges, and the hardest cases |

Order them cheapest first, and only send what survives to expensive scorers. See [LLM as judge](../evals/llm-as-judge.md) for how to keep a judge honest.

**Score the stages, not just the end.** A RAG system needs a retrieval score and an answer score. One combined number cannot tell you whether the passage was missing or the model ignored it, so it cannot tell you what to fix.

## 5. The serving path

```
[trigger: PR, schedule, or manual]
   --> [resolve candidate: prompt version, model version, retrieval config]
   --> [fan out over examples, bounded concurrency]
   --> [scorers, cheapest first]
   --> [aggregate, compare against baseline]
   --> [gate: pass, fail, or warn] --> [store run, per-example diffs]
```

The gate is the point of the system. It compares against the current production baseline and fails on a regression beyond a threshold, per metric, with the option to require a human override.

Two tiers of run: a fast subset on every pull request, and the full suite nightly. A 15-minute gate that runs on 300 examples is more valuable than a two-hour suite nobody waits for.

## 6. Keep it working

- **Judges drift.** When the judge model version changes, every historical score becomes incomparable. Pin the judge version to the run and re-measure agreement after any change.
- **Datasets leak.** A case that gets fixed by tuning on it stops measuring anything. Keep a held-out set that is not used during development.
- **Flaky examples.** Track per-example variance and quarantine the ones that flip randomly, rather than letting them add noise to every run.
- **Grow from incidents.** Every production failure becomes a test case. That is the discipline that makes the suite worth its cost.

## 7. Count the costs

Thousands of runs a week over hundreds of examples, each with a judge call, is a real bill. It is also cheaper than shipping a regression.

Levers: run the fast subset on pull requests. Cache results for unchanged candidate and example pairs. Use a small judge for easy checks, and cap per-team spend with a quota.

## Common follow-ups

- **How do you know the judge is right?** Measure agreement with humans on a couple of hundred cases, and re-measure after any judge change.
- **What goes in the dataset?** Hand-written cases, reviewed production traffic, and every past incident. Include cases where the correct answer is to abstain.
- **How do you stop the score drifting for no reason?** Pin dataset, judge, and sampling settings to the run, and report variance.
- **What blocks a release?** A regression beyond threshold on a gating metric, with a documented override path.

## Common mistakes

- One overall quality score, so a regression cannot be localized.
- Editing datasets in place.
- Judges with no measured agreement against people.
- A suite so slow that teams skip it.

## Go deeper

- Written for a general system design round, architecture first: [Design a model evaluation pipeline](https://github.com/design-gurus/grokking-system-design/blob/main/questions/design-model-evaluation-pipeline.md).
- The full worked solution: [Grokking the AI System Design Interview](https://www.designgurus.io/course/grokking-the-ai-system-design-interview).
