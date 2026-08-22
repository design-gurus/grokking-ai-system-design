# LLM as judge

> Using a model to grade another model's output, and the checks that keep the grade honest.

## What it is

Open-ended output has no single right answer, so exact match does not work. LLM as judge means sending the output, and usually the input and a rubric, to a model and asking it to score.

It is the only practical way to evaluate generation at volume. It is also the part of an evaluation stack most likely to be quietly wrong.

## When to use it

- Scoring thousands of generated answers per day, where human review cannot keep up.
- Regression testing a prompt or model change against a fixed set of cases.
- Ranking two candidate outputs against each other, which judges do far better than absolute scoring.

Do not use it when a cheaper check works. A schema check, a keyword check, or a unit test is faster, free, and deterministic. Use a judge only for the part that genuinely needs judgment.

## How to make it reliable

**Score one narrow thing at a time.** "Rate this answer from 1 to 10" produces noise. "Is every claim in this answer supported by the passages, yes or no" produces a usable signal. Split quality into named checks: grounded, complete, correct format, right tone.

**Prefer pairwise comparison.** Asking which of two answers is better is more stable than asking for an absolute score. The judge no longer has to keep a consistent scale across cases.

**Fix the position bias.** A judge tends to prefer whichever answer it sees first. Run each comparison in both orders and keep only the agreements, or count a disagreement as a tie.

**Give the rubric with examples.** A short rubric with two graded examples beats a long rubric with none.

**Never let a model grade its own output with the same prompt.** Use a different model, or at minimum a separate prompt with a separate role.

## The check that makes it trustworthy

**Measure the judge against people.** Take 100 to 200 cases, have humans grade them, and compute how often the judge agrees. That agreement rate is a property of your judge, and you should know it before you trust any number the judge produces.

If agreement is high, the judge can run at volume and humans review a sample. If agreement is low, the rubric is wrong, not the humans. Rewrite it and measure again.

Re-measure after any change to the judge model or the rubric. A judge upgrade silently changes every score in your history, which makes old and new results not comparable.

## Trade-offs

| Pro | Con |
|-----|-----|
| Scales to thousands of cases per run | Costs tokens on every evaluation run |
| Handles output with no single right answer | Agreement with people must be measured, or the score is unverified |
| Consistent, unlike tired human reviewers | Consistently biased toward longer, more confident answers |
| Fast enough to gate a pull request | A judge model change invalidates your history |

## Known biases to name in an interview

- **Length.** Judges prefer longer answers, even when the short one is correct.
- **Position.** The first option wins more often than it should.
- **Self preference.** A model tends to prefer text that reads like its own.
- **Confidence.** A confident wrong answer scores above a hedged right one.

Naming two of these, and the countermeasure, is the difference between having heard of the technique and having used it.

## What to say in an interview

**The sentence to say.** "I would use a judge model for the open-ended part, scored on named checks rather than one overall rating. First I would measure the judge's agreement with human graders on a couple of hundred cases. Below that agreement bar the judge is not evidence."

## Go deeper

Full lesson on evaluating LLM systems: [Grokking the AI System Design Interview](https://www.designgurus.io/course/grokking-the-ai-system-design-interview).
