# Speculative decoding

> A small model guesses the next few tokens, the large model checks them all in one pass, and the answer is identical to what the large model would have produced.

## What it is

Generation produces one token per model step, and each step reads the entire model from memory. That makes decode slow for a single request, no matter how powerful the GPU is.

Speculative decoding breaks the one-token-per-step rule. A small draft model proposes several tokens ahead. The large target model then verifies all of them in a single forward pass. Checking a sequence is parallel work, while generating one is not.

## The problem it solves

A single generation request uses a small share of a GPU's compute. The limit is memory bandwidth: producing one token requires reading every weight. Adding compute does not help, and neither does a bigger machine.

For a chat product where one user is waiting, this sets the floor on tokens per second. Speculative decoding raises that floor without changing the model or the output.

## How it works

```
draft model:  proposes  the  cat  sat  on  the  mat
target model: verifies  ok   ok   ok   no
result:       accept "the cat sat", correct the fourth token, continue
```

1. The draft model generates k tokens quickly. It is much smaller, so this is cheap.
2. The target model processes all k in one pass and computes what it would have produced at each position.
3. Tokens are accepted while they match. The first mismatch is replaced with the target model's own token, and everything after it is discarded.
4. The loop repeats from there.

The important property: with the correct acceptance rule, the output distribution is exactly the target model's. This is a speed optimization with no quality cost, which is rare enough to be worth saying out loud.

## What decides the gain

**Acceptance rate.** If the draft agrees with the target most of the time, most proposed tokens survive and the speedup is large. If it rarely agrees, you paid for the draft and threw the work away.

Acceptance is high on predictable text: boilerplate, code with obvious structure, quotes from the prompt. It is low on genuinely uncertain content.

**Draft quality against draft cost.** A larger draft agrees more often and costs more per proposal. The usual choices are a much smaller model from the same family, or a cheap non-model predictor for repeated text.

## When to use it

- Latency-bound serving with small batches, where the GPU has spare compute. This is the main case.
- Interactive products where tokens per second is what the user feels.

It helps least when the server is already running large batches. There the GPU is busy and the compute the verification step needs is no longer free.

## Trade-offs

| Pro | Con |
|-----|-----|
| Two to three times faster decode in the good case | The gain depends on acceptance, which varies by content |
| Output is identical to the target model | Two models to load, serve, and keep in memory |
| No retraining, no quality trade | Rejected tokens are wasted compute |
| Composes with continuous batching | Less useful exactly when the batch is already full |

## Numbers to know

- Typical proposal length: 3 to 8 tokens per step.
- Reported speedups are commonly around 2 to 3 times for latency-bound serving.
- The draft model is usually 10 to 20 times smaller than the target.
- The draft's weights take memory that would otherwise hold the KV cache, so it costs concurrency.

## How to talk about it in an interview

The signal is knowing that decode is memory bound, which is why the trick works at all.

**The sentence to say.** "Decode is limited by memory bandwidth, not compute, so there is spare compute at low batch sizes. A small draft model proposes several tokens and the large model verifies them in one pass. The output is identical, and interactive latency improves by roughly two times."

Expect these follow-ups:

- Why the output quality does not change.
- When it stops helping. When batches are large and the GPU is already saturated.
- What else you would do for latency first. Usually shorter prompts, a smaller model, and prefix caching.

## Go deeper

- Related: [continuous batching and the KV cache](continuous-batching-and-kv-cache.md), [quantization and distillation](quantization-and-distillation.md)
- Course: [Grokking the AI System Design Interview](https://www.designgurus.io/course/grokking-the-ai-system-design-interview)
