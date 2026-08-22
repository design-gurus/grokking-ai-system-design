# Continuous batching and the KV cache

> Why LLM serving throughput is a memory management problem, not a compute problem.

## What it is

Generating text is a loop. The model reads the whole prompt once, then produces one token at a time, and each new token depends on every token before it.

To avoid recomputing the earlier tokens on every step, the server keeps their intermediate attention state in GPU memory. That state is the KV cache, short for key-value cache. It makes generation fast, and it grows with every token produced.

Continuous batching is the scheduling trick that keeps the GPU full while requests of very different lengths come and go.

## The problem it solves

A single generation request uses a tiny fraction of a modern GPU. Running one request at a time wastes almost all of the hardware, so servers batch requests together.

Classic static batching waits for a group of requests, runs them together, and returns when all of them finish. That fails badly here, because generation lengths differ wildly. One request that generates 900 tokens holds the whole batch, while nine requests that finished at 20 tokens keep their slots unused. Throughput drops and the fast requests become slow.

The second problem is memory. Each request's KV cache grows as it generates, and the server does not know the final length in advance. Reserving the maximum for every request wastes most of the memory. Reserving too little means a request fails halfway through an answer.

## How it works

**Continuous batching** works at the level of a single model step, not a whole request. After every step, the scheduler removes requests that finished and admits waiting requests into the free slots. The batch is reformed constantly, so the GPU keeps working while individual requests come and go.

```
step 1: [A B C D]
step 2: [A B C D]      C finishes
step 3: [A B E D]      E admitted into C's slot
```

**Paged memory** solves the cache growth. Instead of one contiguous reservation per request, the cache is split into small fixed-size blocks, allocated as the request needs them, like virtual memory pages. Blocks do not need to be next to each other. Two requests that share a prefix, for example the same long system prompt, can point at the same blocks instead of storing two copies. See [PagedAttention and vLLM](../deep-dives/paged-attention-vllm.md).

## Why the two halves of a request differ

| Phase | What happens | Bound by |
|-------|--------------|----------|
| Prefill | The whole prompt is processed at once | Compute |
| Decode | One token is produced per step | Memory bandwidth |

Prefill is one big parallel operation, so it uses the GPU well. Decode reads the entire model's weights from memory to produce one token. It is limited by memory bandwidth, not by arithmetic.

This split explains the two latency numbers users feel. Time to first token is prefill. Tokens per second after that is decode. It also explains why batching helps so much: the weights are read once per step and shared by every request in the batch.

## When to use it

Any time you serve your own model. Every serious open-source serving stack does both of these, so the practical decision is which stack to use, not whether to build this.

You do not need it when you call a hosted API. The provider does it for you, and your levers move to prompt length, caching, and model choice instead.

## Trade-offs

| Pro | Con |
|-----|-----|
| Several times more requests per GPU than static batching | The scheduler is complex, and tail latency depends on its policy |
| Paged blocks nearly remove wasted cache memory | Small per-step bookkeeping cost |
| Shared prefixes cut memory for repeated system prompts | Sharing needs exact prefix matches |
| Long and short requests coexist without blocking each other | Under load, admission becomes a queue, so p99 latency needs its own limit |

## Numbers to know

- Weights at half precision: about 2 bytes per parameter, so an 8 billion parameter model needs about 16 GB.
- The KV cache grows with tokens, layers, and concurrency. On a busy server it often uses more memory than the weights.
- Decode speed for a mid-size model on one high-end GPU: tens of tokens per second per request, with many requests in flight.
- Doubling the batch size raises throughput far more than it raises per-request latency, until memory runs out.

## How to talk about it in an interview

The signal is that you know serving cost is set by memory and scheduling, not by raw compute.

**The sentence to say.** "I would serve behind a runtime that does continuous batching with paged KV cache blocks. The GPU stays full because finished requests are replaced every step, and the shared system prompt is stored once instead of once per request."

Expect these follow-ups:

- What sets time to first token, and what sets tokens per second.
- What happens under a traffic spike. The answer is admission control and a queue with a limit, not unbounded batching.
- How you would cut cost. Route easy requests to a smaller model, cache repeated prefixes, cap output length.

## Go deeper

The full lesson on inference economics and serving optimizations: [Grokking the AI System Design Interview](https://www.designgurus.io/course/grokking-the-ai-system-design-interview).

The platform this sits inside, at general system design altitude: [Design an LLM inference platform](https://github.com/design-gurus/grokking-system-design/blob/main/questions/design-llm-inference-platform.md).
