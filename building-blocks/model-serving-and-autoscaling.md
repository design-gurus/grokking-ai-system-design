# Model serving and autoscaling

> Running model replicas behind a load balancer, where the replicas take minutes to start and cost more per hour than the rest of your fleet combined.

## What it is

Model serving is the service that loads a model into memory and answers prediction requests. Autoscaling is adding and removing replicas as traffic changes.

Both look like ordinary web serving. Two properties make them different: startup is slow, and the hardware is expensive enough that idle capacity is a real budget line.

## The problem it solves

A single replica handles a fixed number of concurrent requests. Beyond that, latency climbs, and the queue grows faster than it drains. Fixed capacity sized for peak wastes money all night. Fixed capacity sized for average fails every morning.

## Why the usual autoscaling signal is wrong

Scaling on CPU utilization is the default in most platforms, and it is a poor signal here.

A GPU replica can sit at low CPU while the GPU is saturated. A replica can also look busy while every request is waiting in a queue. The metrics that reflect reality are:

| Signal | What it tells you |
|--------|-------------------|
| Queue depth, or requests waiting | Demand is above capacity right now |
| Time in queue before processing | The part of latency you control by scaling |
| Batch fullness or GPU utilization | Whether the hardware is actually working |
| Concurrent requests per replica | The number you tuned the replica for |

Scale on queue depth or queue time. Both rise before the user-visible latency does.

## Cold start is the hard part

A new replica must be scheduled, pull a container image of several gigabytes, load model weights of tens of gigabytes, and warm up. That is minutes, not seconds.

So autoscaling cannot be the only answer to a spike. Three defenses, and you should name at least two:

1. **A warm pool.** Keep spare replicas running. Expensive, and the only thing that responds instantly.
2. **Scale up early and down slowly.** Trigger on a low threshold, and keep replicas for a long cool-down period. Flapping is worse than a little waste.
3. **Admission control.** When capacity is gone, return a clear rejection or a degraded answer quickly. A request queued for 40 seconds helps nobody.

## Serving several models on one fleet

Two models rarely fill one GPU evenly. Three patterns:

- **One model per replica.** Simple, isolated, wasteful when a model is idle.
- **Several models per replica.** Better utilization, and one noisy model affects the others.
- **Load on demand, with an eviction policy.** Fits a long tail of rarely used models, at the cost of a load delay on a miss.

For language models, the runtime already batches requests continuously, so per-replica capacity is set by memory rather than by a request count. See [continuous batching and the KV cache](continuous-batching-and-kv-cache.md).

## Trade-offs

| Pro | Con |
|-----|-----|
| Capacity follows traffic, so nights are cheap | Cold start means it follows slowly |
| A warm pool absorbs spikes | You pay for idle expensive hardware |
| Preemptible capacity cuts cost sharply | The provider can take it back mid-request |
| Several models per replica raises utilization | Failure and noisy neighbors are shared |

## Numbers to know

- Container pull plus weight load for a large model: 1 to 5 minutes. Plan around this number.
- Target utilization of 60 to 70 percent leaves room for a spike while a replica starts.
- GPU capacity is the dominant cost in most AI systems, so utilization is a first-class metric, not an operations detail.
- Preemptible or spot capacity is much cheaper, and it fits batch work far better than live serving.

## How to talk about it in an interview

**The sentence to say.** "I would scale on queue time rather than CPU, because a GPU replica can be saturated at low CPU. Cold start is minutes, so I keep a warm buffer and scale up early. Under overload I shed load with a fast rejection, rather than a long queue."

Expect these follow-ups:

- What the user sees when capacity runs out. A cached or simpler answer, or a clear error, never a 40 second wait.
- How you would use cheaper capacity. Batch and offline jobs on preemptible machines, live serving on reserved ones.
- How you size one replica. Load test it to find the concurrency where latency starts to climb.

## Go deeper

- Sizing the fleet: [GPU capacity planning](gpu-capacity-planning.md)
- Course: [Grokking the AI System Design Interview](https://www.designgurus.io/course/grokking-the-ai-system-design-interview)
