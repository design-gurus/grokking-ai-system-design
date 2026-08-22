# Design [system]

> One-line description of what the system does and who uses it.

**Family:** Classic ML | GenAI and LLM product | Agentic
**Also asked as:** other phrasings of the same question.

## 1. Frame the problem

What the product does, and the machine learning task in one sentence. State the scale: users, requests per second, catalog size.

## 2. Define success

| Metric type | Metric | Target |
|-------------|--------|--------|
| Offline | | |
| Online | | |
| Guardrail | | |

One sentence on why the offline metric is a filter and the online metric is the decision.

## 3. Map the data

Where labels come from, and how long they take to arrive. The feature groups. How training and serving read the same values. For retrieval systems, the corpus, the chunking, and how permissions travel.

## 4. Pick the model

The baseline first, then the upgrade and what it buys. Name the cost of the upgrade in latency or money.

## 5. The serving path

State the latency budget, then spend it.

```
[client] --> [retrieval] --> [features] --> [model] --> [response]
```

What happens when the model is slow or unavailable.

## 6. Keep it working

Monitoring, drift, retraining trigger, rollout, and rollback. The feedback loop, and how it can poison itself.

## 7. Count the costs

The estimate, then the lever you would pull.

## Common follow-ups

- The two or three questions an interviewer always asks next, answered in a line each.

## Common mistakes

- The mistakes that cost candidates this question.

## Go deeper

The full worked solution: [Grokking the AI System Design Interview](https://www.designgurus.io/course/grokking-the-ai-system-design-interview).
