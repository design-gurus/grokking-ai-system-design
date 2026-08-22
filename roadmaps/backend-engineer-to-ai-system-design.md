# Backend engineer to AI system design

> For engineers who have shipped real services and are now facing a round with a model in it. Three to four weeks, about an hour a day.

## Start here: what actually transfers

Most of what you know transfers. Load balancing, caching, queues, sharding, replication, and idempotency all appear in AI systems unchanged. A serving path is still a request path with a latency budget.

Four things do not transfer, and they are the whole gap:

1. **Quality is a number, not a spec.** You will be asked to choose a metric and defend it.
2. **Data is upstream of correctness.** Where labels come from decides whether the system can work at all.
3. **Behavior changes with no deploy.** Models get worse on their own, so monitoring is part of the design.
4. **Cost per request is large and visible.** Tokens and GPUs are a design input, not an operations detail.

You do not need to learn model architectures, math, or training tricks. Interviewers in this round are hiring a systems engineer who is not afraid of a model.

## Week 1: the frame and the vocabulary

Goal: be able to run all seven steps out loud, badly. Fluency comes later.

- Read the [seven-step framework](../cheat-sheets/ai-system-design-framework.md) twice. Write the seven step names on paper from memory.
- Read the [glossary](../glossary.md). Stop at any term you could not define to a colleague.
- Read [what the three families are](../README.md#the-three-question-families) and place three questions you have heard of into them.
- Read [numbers you should know](../cheat-sheets/ai-numbers.md) and memorize the five anchors.

**Exercise.** Take any system you have built at work. Add a model to it in your head, then walk the seven steps out loud in 20 minutes. It will be bad. Do it anyway, because it exposes which step you have nothing to say about.

## Week 2: the data half

This is the week that closes the largest part of the gap, because it is the half your backend experience does not cover.

- [Feature stores](../building-blocks/feature-stores.md). Understand training and serving skew well enough to explain it without notes.
- Read [evaluation](../evals/) end to end. Learn the offline, online, and guardrail split.
- Work through [design feed ranking](../questions/design-feed-ranking.md). It is the reference question for the classic ML family.

**Exercise.** For three products you use daily, say where the training labels come from and how long they take to arrive. A refund fraud label takes 60 days. A click label takes a second. That delay changes the entire design.

## Week 3: the LLM half

- [RAG vs fine-tuning vs prompting](../cheat-sheets/rag-vs-fine-tuning-vs-prompting.md). This decision appears in almost every GenAI question.
- [Vector indexes](../building-blocks/vector-indexes.md), including the filtering trap.
- [Continuous batching and the KV cache](../building-blocks/continuous-batching-and-kv-cache.md), then [PagedAttention](../deep-dives/paged-attention-vllm.md). This is where your systems background gives you an advantage over candidates with an ML background.
- Work through [design enterprise document Q and A](../questions/design-enterprise-document-qa.md).

**Exercise.** Estimate the monthly token bill for a feature you would actually build at your company. Then name the lever you would pull to halve it.

## Week 4: agents, then practice out loud

- Read [design a customer support agent](../questions/design-a-customer-support-agent.md), paying attention to the four bounds on the loop.
- Do three timed 45-minute walkthroughs, one from each family, spoken out loud and recorded.
- Listen to one recording. You are checking for two failures: did you reach step 7, and did you name a metric before designing the system.

## The four sentences to have ready

An AI round rewards a small number of specific sentences. Have these ready, in your own words:

- "The task is to predict X given Y, and here is what one example looks like."
- "Offline I would measure A, online I would measure B, with C as a guardrail."
- "Training and serving must read the same feature definition, or the model sees a different world in production."
- "At this volume the bill is roughly N per month, and the lever I would pull first is M."

## Where you have an edge

Candidates from a research background usually describe the model well and stop. Your advantage is the serving path, the failure modes, the rollback plan, and the cost. Say those things without being asked, and the round goes well.

## Go deeper

The course covers all of this with worked solutions and eight timed scenario walkthroughs: [Grokking the AI System Design Interview](https://www.designgurus.io/course/grokking-the-ai-system-design-interview).
