# Glossary

The vocabulary of an AI system design interview, in one line each. If a term below is new to you, learn it before the round. Interviewers use these words without explaining them.

## The task

**Model.** A function learned from data. It takes features or text and returns a prediction or generated output.

**Inference.** Running the model to get an output. Also called serving or prediction.

**Training.** Fitting the model to data. Happens offline, on a schedule or a trigger.

**Fine-tuning.** Training an existing model further on your own examples, so new behavior lives in the weights.

**Prompt.** The instructions and context sent with a request to a language model.

**Token.** The unit a language model reads and writes. Roughly 1.3 tokens per English word.

**Context window.** The maximum number of tokens a model can read in one request, including the prompt and the conversation.

## Data

**Feature.** A number the model reads as input, for example a user's order count in the last 30 days.

**Label.** The correct answer for a training example. Often produced by user behavior, sometimes by human review.

**Label delay.** The time between an event and the arrival of its label. A click label takes seconds. A chargeback label takes weeks.

**Feature store.** A system that computes features once and serves them to both training and production. See [feature stores](building-blocks/feature-stores.md).

**Training and serving skew.** The model sees different feature values in production than in training. The most common silent failure in machine learning.

**Point-in-time join.** Building a training row with the feature values as they were at the moment of the event, so future information does not leak into training.

**Data leakage.** Any case where training data contains information that would not exist at prediction time. Produces excellent offline scores and a failure in production.

## Retrieval

**Embedding.** A list of numbers representing an item, arranged so that similar items are close together.

**Vector index.** The structure that finds nearest vectors quickly. See [vector indexes](building-blocks/vector-indexes.md).

**ANN.** Approximate nearest neighbor. Search that trades a little accuracy for a large speed gain.

**Recall at k.** The share of the true top k results that your search actually returned. The main quality knob in retrieval.

**Chunking.** Splitting documents into passages small enough to embed and to fit in a prompt.

**RAG.** Retrieval-augmented generation. Search a corpus, then paste the best passages into the prompt.

**Reranking.** A second, slower, more accurate model that reorders a small candidate list.

**Hybrid search.** Running keyword search and vector search together, because each finds what the other misses.

**Grounding.** Requiring every claim in an answer to come from a retrieved source.

## Serving

**KV cache.** The stored attention state for the tokens generated so far, which is what makes generation fast and memory hungry.

**Prefill.** Processing the whole prompt at once, before the first output token. Sets time to first token.

**Decode.** Producing output tokens one at a time. Limited by memory bandwidth.

**Continuous batching.** Admitting and retiring requests every model step, so the GPU stays busy. See [continuous batching](building-blocks/continuous-batching-and-kv-cache.md).

**Quantization.** Storing weights in fewer bits to shrink the model, at a measurable quality cost.

**Distillation.** Training a small model to copy a large one.

**Speculative decoding.** A small model proposes several tokens, and the large model checks them in one step.

**Time to first token.** How long the user waits before text starts appearing. Usually the latency number that matters.

## Quality

**Offline metric.** A score on held-out data, computed before launch. A filter, not a decision.

**Online metric.** A score measured on real users in an experiment. The thing that decides a launch.

**Guardrail metric.** The number you refuse to trade away, for example p99 latency or reported harmful content.

**AUC.** Area under the ROC curve. A ranking quality score between 0.5 (random) and 1.0.

**NDCG.** A ranking metric that rewards putting the best items highest.

**Calibration.** Whether a predicted probability of 0.3 actually happens 30 percent of the time. Matters whenever the score feeds a downstream calculation, like an ad auction.

**Drift.** The world changes and the model's inputs or outputs shift away from what it was trained on.

**LLM as judge.** Using a model to grade another model's output. See [LLM as judge](evals/llm-as-judge.md).

**Hallucination.** A fluent, confident, wrong output. Retrieval and grounding reduce it. Nothing removes it.

## Agents

**Agent.** A loop where a model chooses a tool, reads the result, and chooses again until it stops.

**Tool calling.** Letting a model invoke a defined function with structured arguments.

**Trajectory.** The full sequence of steps an agent took. Evaluated separately from the final answer.

**Bounds.** The limits that stop the loop: a step limit, a time limit, a cost limit, and repeat detection.

**Prompt injection.** Input crafted to make the model ignore its instructions. Defended in the tools and permissions, never in the prompt.

**Human in the loop.** Routing uncertain or high-value cases to a person before the action takes effect.

## Experiments

**A/B test.** Splitting users into groups to measure the effect of a change.

**Shadow mode.** Running a new model on live traffic without showing its output, to compare it safely.

**Canary.** Sending a small share of traffic to a new model before a full rollout.

**Interleaving.** Mixing results from two rankers in one list, which detects small ranking differences faster than an A/B test.

**Exploration.** Deliberately showing some items the model would not have chosen, so the training data is not limited to what the model already prefers.
