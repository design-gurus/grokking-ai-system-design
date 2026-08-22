# Design an LLM inference platform

> A shared service that serves many models to many teams on a fixed pool of GPUs, where memory sets the capacity and one tenant can starve the rest.

**Family:** Platform
**Also asked as:** design a model serving platform, design an internal inference service.

## 1. Frame the problem

Product teams want to call a model. They should not each buy GPUs, learn a serving runtime, and build their own autoscaling.

The platform's job: take a model and a request, and return tokens. Inside a stated latency target, at a known cost per token, without one team's traffic hurting another's.

Scale to assume: 20 internal teams, 12 models, a pool of 100 GPUs, 5,000 requests per second at peak with very uneven sizes.

## 2. Define success

| Metric type | Metric | Target |
|-------------|--------|--------|
| Service | Time to first token, and tokens per second, per tier | Stated per tier, measured at p95 |
| Efficiency | GPU utilization, tokens served per GPU hour | Utilization 60 to 70 percent |
| Fairness | Latency for a small tenant while a large one is at peak | Within the tier's target |
| Cost | Cost per million tokens, attributed per team | Falling over time |
| Adoption | Share of company model calls on the platform | Rising |

Time to first token and tokens per second are separate numbers because they come from different parts of the request. Prefill sets the first, decode sets the second. See [continuous batching and the KV cache](../building-blocks/continuous-batching-and-kv-cache.md).

## 3. Map the data

What flows through the platform, and what has to be stored:

- **Model artifacts.** Weights, tokenizer, and serving config, versioned in a registry. See [model registry and rollout](../building-blocks/model-registry-and-rollout.md).
- **Requests and responses.** Logged with model version, token counts, latency, and the calling team. This is what makes cost attribution and later evaluation possible.
- **Tenant configuration.** Quotas, priority tier, allowed models.
- **Metrics.** Queue depth, batch fullness, memory use, per model and per replica.

Logging prompts is a privacy decision, not a default. Offer per-tenant control, redact known sensitive fields, and set a retention window.

## 4. The core design choice

Three decisions carry this design.

**One, the serving runtime.** Use one that does continuous batching with a paged KV cache. Reforming the batch every model step is what keeps the GPU busy when request lengths differ by 50 times. Writing this yourself is not the answer.

**Two, how models share hardware.**

| Approach | Fits |
|----------|------|
| Dedicated replicas per model | High-traffic models, predictable load |
| Several small models on one GPU | Low-traffic models, better utilization |
| Load on demand with eviction | A long tail of rarely used models |

A large model that does not fit on one GPU is split across several. Those GPUs then become a single unit that must be scheduled together.

**Three, tiers instead of one queue.** Interactive requests, batch requests, and background jobs have different targets. Give them separate queues and separate admission rules. Batch work then fills the gaps that interactive traffic leaves, which is where the utilization gain comes from.

## 5. The serving path

```
[request] --> [gateway: auth, quota, tier]
          --> [router: which model, which replica pool]
          --> [queue for that tier]
          --> [replica: continuous batching]
          --> [stream tokens back]
```

**Admission control is the part candidates miss.** When memory for the KV cache is exhausted, the correct behavior is to reject quickly or queue with a stated limit. Accepting everything produces a queue where every request misses its target, which is worse than serving fewer requests well.

**Streaming must be end to end.** Every hop has to pass tokens through without buffering, or the first-token number the user feels is destroyed by the platform itself.

## 6. Keep it working

- **Cold start is minutes**, dominated by loading weights. Keep a warm pool per popular model and scale up early. See [model serving and autoscaling](../building-blocks/model-serving-and-autoscaling.md).
- **Noisy neighbors.** One tenant sending 100,000-token prompts consumes the cache that everyone else needs. Cap prompt length per tier and count tokens, not requests, against quotas.
- **Model rollout.** New versions go to a small share of traffic first, and the previous version stays loadable.
- **Watch queue time separately from model time.** They have different fixes: capacity for the first, optimization for the second.

## 7. Count the costs

The fleet is the cost, so utilization is the metric that pays for the platform. Idle GPUs are the largest waste in most AI infrastructure.

Do the arithmetic out loud: peak output tokens per second, divided by measured tokens per second per GPU, plus headroom to 65 percent utilization. See [GPU capacity planning](../building-blocks/gpu-capacity-planning.md).

Levers: fill troughs with batch work. Quantize models so more fit per GPU. Run preemptible capacity for the batch tier, and route easy requests to smaller models.

## Common follow-ups

- **How do you stop one team starving the others?** Per-tenant token quotas, tiered queues, prompt length caps, and admission control.
- **What happens when a request cannot be served?** A fast rejection with a retry hint, never an unbounded wait.
- **How do you decide dedicated against shared replicas?** By traffic volume and latency tier.
- **How do you attribute cost?** Tokens counted at the gateway, tagged by team, priced per model.

## Common mistakes

- Designing around request count rather than token count and memory.
- No admission control, so overload becomes universal slowness.
- Ignoring cold start when describing autoscaling.
- Buffering the stream somewhere in the platform.

## Go deeper

- Written for a general system design round, architecture first: [Design an LLM inference platform](https://github.com/design-gurus/grokking-system-design/blob/main/questions/design-llm-inference-platform.md).
- The full worked solution: [Grokking the AI System Design Interview](https://www.designgurus.io/course/grokking-the-ai-system-design-interview).
