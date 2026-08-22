# GPU capacity planning

> Turning a traffic estimate into a number of GPUs, starting with memory rather than speed.

## What it is

Capacity planning answers two questions an interviewer will ask at step 7: how many machines does this need, and what does that cost per month.

For AI systems the arithmetic is short, and doing it out loud is worth more than getting it exactly right.

## Memory first

A GPU cannot serve a model it cannot hold. Memory is the first constraint and it is the one that decides the hardware.

```
weights   = parameters x bytes per parameter
KV cache  = grows with tokens in flight, so with concurrency and conversation length
free      = GPU memory - weights - overhead
```

At 2 bytes per parameter, an 8 billion parameter model needs about 16 GB and a 70 billion parameter model about 140 GB. On an 80 GB GPU the first leaves roughly 60 GB for the cache. The second does not fit at all until it is [quantized](quantization-and-distillation.md) or split across GPUs.

**Whatever memory is left after the weights is your concurrency budget.** That is the sentence that connects memory to throughput, because every request in flight holds cache.

## Then throughput

```
GPUs = peak tokens per second / tokens per second per GPU
```

Both numbers come from measurement, not from a specification sheet. Get tokens per second per GPU from a load test at your batch size and sequence length. Get peak tokens per second from traffic:

```
peak requests per second x output tokens per request = peak output tokens per second
```

Prefill and decode load the machine differently. A long prompt with a short answer and a short prompt with a long answer need different capacity, at the same request count. See [continuous batching and the KV cache](continuous-batching-and-kv-cache.md).

## Peak, not average

Traffic is not flat. A business tool has a working-hours peak several times its daily average, and a consumer product peaks in the evening.

Size for peak, then decide what to do with the trough:

- Run batch and offline work at night on the same fleet.
- Use cheaper preemptible capacity for anything that can be interrupted.
- Scale down, accepting that scaling back up takes minutes. See [model serving and autoscaling](model-serving-and-autoscaling.md).

Never size for the average and hope. Queueing does not smooth a sustained peak, it only delays it.

## Leave headroom

A fleet planned at 100 percent utilization fails at 85. Queues grow non-linearly as utilization approaches saturation, and latency goes with them.

Plan for 60 to 70 percent target utilization. The gap absorbs spikes and gives new replicas time to start.

## Self-hosted against hosted

Do this comparison out loud, because it is the real decision behind the number.

| | Hosted API | Self-hosted GPUs |
|---|---|---|
| Cost shape | Per token, scales with usage | Per hour, whether busy or idle |
| Cheaper when | Traffic is low or spiky | Traffic is high and steady |
| Effort | A client library | Serving, scaling, upgrades, on-call |
| Control | Provider's models and limits | Any model, any version, data stays in your account |

The crossover is about utilization. A reserved GPU only beats per-token pricing when it is kept busy. Sustained, predictable load favors self-hosting. Bursty load favors the API.

## Numbers to know

- 2 bytes per parameter at half precision, 1 byte at 8-bit.
- 80 GB is the usual high-end data center GPU. A 70B model needs two of them at half precision.
- Target 60 to 70 percent utilization.
- Cold start for a large model is 1 to 5 minutes, which is why headroom matters more than fast scaling.
- Preemptible capacity is substantially cheaper and can be taken back at any moment.

## How to talk about it in an interview

**The sentence to say.** "Memory first: the weights take 16 GB, so the rest of the 80 GB is the concurrency budget. Peak is 200 requests per second at 300 output tokens, so 60,000 output tokens per second. At measured throughput per GPU that gives the fleet size, and I would plan to 65 percent utilization because cold start is minutes."

Expect these follow-ups:

- Whether to host or call an API, and at what utilization that flips.
- What runs on the fleet at 3 in the morning.
- What happens when the peak is twice what you planned.

## Go deeper

- The anchors: [numbers you should know](../cheat-sheets/ai-numbers.md)
- Course: [Grokking the AI System Design Interview](https://www.designgurus.io/course/grokking-the-ai-system-design-interview)
