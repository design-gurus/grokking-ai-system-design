# The AI system design framework

> Seven steps that fit any AI design question, with timings for a 45-minute round.

A classic framework stops at architecture. This one keeps going into data, evaluation, and cost. Those three steps are what an AI round adds. Use the same seven steps for a ranking system, a chatbot, and an agent. Only the weight of each step changes.

## The seven steps

| Step | Minutes | The one thing to produce |
|------|---------|--------------------------|
| 1. Frame the problem | 5 | One sentence naming the prediction or the generation task |
| 2. Define success | 5 | An offline metric, an online metric, and a guardrail |
| 3. Map the data | 8 | Where labels come from, and how features reach serving |
| 4. Pick the model | 5 | A simple baseline, then the upgrade and why |
| 5. The serving path | 10 | One request walked end to end inside a latency budget |
| 6. Keep it working | 7 | Monitoring, retraining trigger, and rollback |
| 7. Count the costs | 5 | Cost per request and cost per month |

Five minutes are left for questions. If you are running late, never drop steps 2 and 7. Those two separate senior candidates from junior ones.

## Step 1: Frame the problem

Turn the product ask into a machine learning task. "Design a feed" is not a task. "Rank the 500 candidate posts a user could see, by probability of a meaningful interaction in the next session" is a task.

Say out loud:

- Who the user is, and what they see.
- What one prediction or one generated output looks like.
- The scale: users, requests per second, and items in the catalog.

**The sentence to say.** "Before I design anything, let me state the task: given a user and a candidate item, predict the probability of X. Everything else follows from that."

## Step 2: Define success

Name three numbers. Most candidates name none.

| Metric type | What it is | Example |
|-------------|------------|---------|
| Offline | Measured on held-out data, before launch | AUC of 0.78, recall at 100 of 0.9 |
| Online | Measured on real users, in an experiment | Session time up 3 percent |
| Guardrail | The thing you refuse to trade away | p99 latency under 300 ms, reported harmful content flat |

An offline metric is a filter, not a decision. It tells you which model is worth an experiment. Only the online metric decides a launch.

**The sentence to say.** "Offline I would track recall at 100 for retrieval and AUC for ranking. Online I would run an A/B test on session time, with latency and reported content as guardrails."

## Step 3: Map the data

This is the longest step in a classic ML question, and the one that separates real experience from reading.

Answer four questions:

1. **Where do labels come from?** User clicks, purchases, human review, or a rule. Say the delay: a click label arrives in seconds, a fraud chargeback label arrives in 60 days.
2. **What are the features?** Group them: user features, item features, and interaction features that only exist at request time.
3. **How are features computed?** Batch jobs for slow features, a stream for fast ones.
4. **How does serving get the same values?** A [feature store](../building-blocks/feature-stores.md) with one definition used by both training and serving. Different code paths in each is called training and serving skew, and it is the most common production bug in ML.

For a retrieval or LLM question, the same four questions apply to documents. Where the corpus comes from, how it is chunked, how often it changes, and how permissions travel with it.

## Step 4: Pick the model

Interviewers are not testing whether you know the newest architecture. They are testing whether you can pick the smallest thing that works, and say what you would try next.

A safe ladder for ranking starts with logistic regression or gradient-boosted trees. Next is a neural ranker. Then a two-tower model for retrieval, with a heavier ranker on top.

A safe ladder for LLM products: prompt an existing model first, then add retrieval, then fine-tune only when you can name what fine-tuning fixes. See [RAG vs fine-tuning vs prompting](rag-vs-fine-tuning-vs-prompting.md).

**The sentence to say.** "I would ship gradient-boosted trees on 20 features to get a baseline in production. Then I would measure whether a neural ranker beats it, before paying for the extra latency."

## Step 5: The serving path

Draw the request. State the budget first, then spend it.

A 300 ms budget for a ranked feed might spend 20 ms on retrieval, 40 ms on feature fetch, and 60 ms on scoring. The rest is slack. A 3 second budget for a generated answer spends most of its time in the model. Streaming shows the first token in under a second.

Cover:

- Retrieval and candidate generation, when the catalog is too big to score.
- Batch or online inference, and why. [Batch is cheaper, online is fresher](../building-blocks/feature-stores.md).
- What happens when the model is slow or down. A cached answer, a simpler model, or a default ranking. Never an error page.

## Step 6: Keep it working

Models get worse without anyone touching them. Conditions change and the training data stops matching them. That is called drift.

Say what you monitor:

- **Input drift.** The distribution of a feature changes. Cheap to detect, catches most problems.
- **Output drift.** The score distribution shifts, or the answer length doubles.
- **Quality.** Labels arrive late, so real quality is always measured behind live traffic.

Then say the response: retrain on a schedule, retrain on a trigger, and keep the previous model ready so a rollback is one config change. Every model rollout is a canary or a shadow first, never a full switch.

## Step 7: Count the costs

Cost is a design input in this round, not an afterthought. A design that ignores it reads as junior.

For classic ML, count training runs, feature storage, and inference machines. For LLM products, count tokens. See [numbers you should know](ai-numbers.md) for the anchors.

Do it out loud, roughly:

> 1 million requests per day, 800 input tokens and 200 output tokens each. That is about 1 billion tokens a day. At a dollar per million input tokens, this is a five-figure monthly bill before caching.

Then name one lever: a smaller model for easy requests, a cache for repeated questions, or shorter prompts.

## What each level sounds like

| Level | The tell |
|-------|----------|
| Junior | Names models and libraries, skips data and cost |
| Mid | Complete architecture, one metric, no guardrail |
| Senior | Picks a baseline on purpose, states the latency budget, names the cost lever |
| Staff | Argues about what the metric should be, and says which part of the system is allowed to fail |

## Go deeper

Full worked examples of all seven steps, across eight timed scenarios: [Grokking the AI System Design Interview](https://www.designgurus.io/course/grokking-the-ai-system-design-interview).
