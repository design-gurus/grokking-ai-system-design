# Guardrails and safety filters

> Checks around the model that block bad input and bad output, sitting where the model cannot argue with them.

## What it is

A guardrail is a check outside the model. It runs on the way in, on the way out, or both, and it can block, redact, or rewrite.

Instructions inside the prompt are not guardrails. They are requests. A user's message can talk a model out of an instruction, which is called prompt injection. It cannot talk a filter out of anything.

## The problem it solves

Three failures that a product cannot ship with:

- The model produces content that is harmful, defamatory, or against policy.
- The model reveals data the user should not see, or leaks its own system prompt.
- The model is manipulated into taking an action, for example approving something it should refuse.

Each needs a different check, in a different place.

## The layers

```
[user input] --> [input filters] --> [model] --> [output filters] --> [user]
                                        |
                                   [tool limits]
```

**Input filters.** Detect known attack patterns, off-policy requests, and personal data that should be redacted before it reaches a provider. Cheap, fast, and they catch the obvious cases only.

**Output filters.** Classify the generated text before it is shown: harmful content, personal data, competitor claims, legal risk. This is the layer that matters most, because it checks what the user will actually see.

**Tool limits.** For anything that acts, the real guardrail is in the tool: a refund cap, a permission check, an approval threshold. See [tool calling](tool-calling.md).

**Retrieval scoping.** In a RAG system, the strongest privacy guardrail is that the passage never enters the prompt. Permission filtering in the index beats any filter downstream.

## Choosing the check

| Check | Cost | Catches | Misses |
|-------|------|---------|--------|
| Rules and pattern matching | Free | Known strings, formats, obvious cases | Anything phrased differently |
| Small classifier | A few ms | Categories it was trained on | Novel phrasing, subtle context |
| Model as judge | 100s of ms, real money | Nuanced policy violations | Whatever the rubric does not describe |
| Human review | Slow, expensive | Everything, eventually | Volume |

Layer them cheapest first. Rules screen the obvious, a classifier handles volume, and a judge model handles the small share that is genuinely ambiguous. Human review takes the top of that pile.

## The two costs people forget

**Latency.** An output filter cannot run until the text exists, so it fights against streaming. You can stream and check in chunks, accepting that a blocked answer is retracted after the user has seen part of it. Or you buffer, and add the filter's full latency to the first token. Both are defensible. Not having decided is not.

**False positives.** A filter that blocks 2 percent of legitimate answers is a product problem, and it will be felt more than the harm it prevents. Measure the false positive rate on ordinary traffic, not just recall on adversarial cases.

## Measuring guardrails

Guardrails are models and rules, so they need evaluation like any other. Build two sets: a set of genuinely bad inputs, including jailbreak attempts, and a set of ordinary traffic that must pass. Track catch rate on the first and false positive rate on the second.

Re-run both when the base model changes. A model upgrade changes what gets through.

## Trade-offs

| Pro | Con |
|-----|-----|
| The check cannot be argued out of | Every layer adds latency and cost |
| Layers cover each other's misses | False positives block legitimate answers |
| Filters can be updated without touching the model | An attacker who learns the filter can phrase around it |
| Tool limits make harmful actions impossible, not unlikely | Too many refusals make the product feel broken |

## Numbers to know

- A small classifier adds single-digit to low tens of milliseconds. A judge model adds hundreds.
- Set an explicit false positive budget, for example under 1 percent of ordinary traffic blocked.
- Safety checks belong in the [model gateway](model-gateway-and-routing.md) so every product gets them without reimplementing.

## How to talk about it in an interview

**The sentence to say.** "Prompt instructions are not a control, because a user message can override them. I would run a cheap input filter and a classifier on the output before it is shown. The real limits go in the tools, where the model cannot reach them. I would measure both catch rate and false positive rate on normal traffic."

Expect these follow-ups:

- How output filtering works with streaming.
- What stops prompt injection from a retrieved document, not just from the user. The same answer: limits in tools, and treating retrieved text as untrusted.
- What you do when the filter blocks something legitimate.

## Go deeper

- Related: [human in the loop](human-in-the-loop.md), [tool calling](tool-calling.md)
- Course: [Grokking the AI System Design Interview](https://www.designgurus.io/course/grokking-the-ai-system-design-interview)
