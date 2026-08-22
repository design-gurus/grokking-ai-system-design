# Evaluation

The step candidates skip and interviewers grade hardest.

In a classic system design round, correctness is defined by a spec. Here it is a number you choose, defend, and keep measuring. "How do you know it works?" is not a closing question in an AI interview. It is the middle of the conversation, and a vague answer ends the round.

## The three layers

Every AI system needs all three. Naming them in order is a senior signal.

| Layer | What it is | When it runs | What it decides |
|-------|-----------|--------------|-----------------|
| Offline | A fixed labeled set, scored automatically | Every change, in CI | Which candidate is worth an experiment |
| Online | Real users, split into groups | Before a launch | Whether it actually ships |
| Guardrail | The metric you refuse to trade away | Always | Whether to roll back |

An offline win is a filter, not a decision. Offline sets are small, stale, and never contain the case that breaks in production. An online experiment is the only thing that decides a launch, and it costs weeks. That is why the offline layer exists: it is the cheap gate that stops bad candidates from spending experiment time.

## Choosing the offline metric

Match the metric to the task, not to habit.

| Task | Metric that fits | The trap |
|------|-----------------|----------|
| Retrieval | Recall at k, then precision at k after reranking | Measuring accuracy on a task with no fixed answer set |
| Ranking | AUC, normalized entropy, NDCG | Ignoring position bias in the logged data |
| Classification with rare positives | Precision and recall at a chosen threshold | Reporting accuracy, which is meaningless at 1 percent positives |
| Generation, format | Exact match against a schema | Grading the whole answer when only the format matters |
| Generation, open-ended | A judge model plus human spot checks | Trusting the judge without measuring the judge |
| Agents | Task success plus trajectory quality | Grading only the final answer |

## Contents

| Guide | What it covers | Status |
|-------|----------------|--------|
| [LLM as judge](llm-as-judge.md) | Using a model to grade output, and how to know when it is lying to you | Written |
| Offline metrics by task | The full table above, with worked examples | Planned |
| Online experiments and guardrails | A/B tests, interleaving, the guardrail hierarchy | Planned |
| RAG evaluation | Separating retrieval quality from answer quality | Planned |
| Agent trajectory evaluation | Golden trajectories, sandboxes, scoring the path | Planned |
| Red teaming and safety evaluation | Adversarial input, jailbreaks, measuring refusal quality | Planned |

## The rule that covers most cases

**Measure each stage separately, then measure the whole.** A RAG system with one quality score cannot tell you whether the passage was missing or the model ignored it. So it cannot tell you what to fix next. An agent with only an outcome score cannot tell you that it used six tool calls to get there.

## Go deeper

Full lessons on evaluating models, LLM systems, and agent trajectories: [Grokking the AI System Design Interview](https://www.designgurus.io/course/grokking-the-ai-system-design-interview).
