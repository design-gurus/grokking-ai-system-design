# Design a GPU cluster scheduler

> Allocate a fixed pool of very expensive machines across training jobs that need many GPUs at once, and serving jobs that must never be interrupted.

**Family:** Platform
**Also asked as:** design a training cluster, design an ML job scheduler.

## 1. Frame the problem

Decide which job runs on which GPUs, and when. The scheduler exists because GPUs are the scarcest resource in the company and idle ones are pure loss.

Two properties make this different from scheduling ordinary containers:

1. **Distributed training is all or nothing.** A job that needs 32 GPUs cannot start with 30. Giving it 30 wastes them entirely while it waits.
2. **Placement changes performance.** GPUs connected by a fast interconnect inside one machine, or one rack, train far faster than GPUs spread across the cluster. Topology is a correctness-of-cost issue, not a preference.

Scale to assume: 1,000 GPUs, 30 teams, a mix of multi-day training runs, short experiments, and always-on inference.

## 2. Define success

| Metric type | Metric | Target |
|-------------|--------|--------|
| Efficiency | Cluster utilization, and the share of allocated GPUs actually computing | Utilization above 70 percent |
| Fairness | Wait time by team and by job size, against each team's share | No team starved |
| Responsiveness | Queue time for a short interactive job | Minutes, not hours |
| Reliability | Jobs lost to preemption without a usable checkpoint | Near zero |

**Allocated is not the same as used.** A job holding 8 GPUs at 15 percent utilization is a waste the allocation numbers hide. Measure both, and report the gap.

## 3. Map the data

- **Job requests.** GPU count, memory per GPU, expected duration, priority, team, and whether the job can be preempted.
- **Cluster state.** Every GPU, its memory, its health, and the topology that connects it.
- **Quotas.** Each team's guaranteed share and its burst limit.
- **History.** Past runtimes per job type, which is how you predict duration for jobs whose owners always underestimate.

The declared duration is unreliable, so treat it as a hint and use history for planning.

## 4. The core design choice

**Gang scheduling.** A distributed job's GPUs are allocated as one unit or not at all. Without it, partially started jobs hold resources and deadlock the cluster.

**A priority ladder with preemption.**

| Tier | Example | Can be preempted |
|------|---------|------------------|
| Production inference | Live serving | Never |
| High priority training | A launch-blocking model | Rarely |
| Normal training | Ordinary experiments | Yes, with checkpointing |
| Best effort | Sweeps, batch scoring | Freely |

Preemption is only humane if checkpointing works. The scheduler should signal the job, allow a grace period to write a checkpoint, then reclaim. A job that cannot checkpoint should declare it and be scheduled as non-preemptible, at a higher quota cost.

**Backfill.** While a large job waits for enough free GPUs, run short jobs in the gap. They have to finish before the large job's resources are ready. This is where a large part of the utilization gain comes from.

**Fair sharing over time**, not per instant. A team below its share for a week gets priority over a team that has been above it. That stops one team saturating the cluster with a queue of small jobs.

## 5. The serving path

```
[submit] --> [validate: quota, image, resource shape]
         --> [queue, ordered by priority and fair share]
         --> [scheduler loop]
               find a topology-aware placement for the full gang
               otherwise reserve capacity and backfill smaller jobs meanwhile
         --> [launch] --> [monitor] --> [release or preempt]
```

The reservation step is what stops large jobs starving. Without it, a 64-GPU job never runs, because small jobs always fit into the gaps first.

## 6. Keep it working

- **Fragmentation is the slow failure.** Free GPUs scattered across many machines cannot host one large job. Defragment by draining, and prefer placements that keep large contiguous blocks free.
- **Hardware fails.** Detect unhealthy GPUs, drain them, and reschedule the affected job from its last checkpoint.
- **Zombie jobs.** A job that hangs holds GPUs at zero utilization. Alert on allocated-and-idle and reclaim after a grace period.
- **Publish the queue.** Teams behave better when they can see wait times and their own share.

## 7. Count the costs

The cluster is the cost, and it is fixed. Every percentage point of utilization is real money, which is why backfill, preemption, and idle detection all pay for themselves.

Compare against the alternative out loud: renting on demand is more expensive per hour and needs no scheduler. A cluster is worth it when sustained utilization is high.

## Common follow-ups

- **Why gang scheduling?** Because a partially allocated distributed job wastes everything it holds.
- **How do you stop large jobs starving?** Reservation with backfill.
- **How do you handle a job that cannot checkpoint?** It declares itself non-preemptible and pays for that in quota.
- **Why does placement matter?** Interconnect bandwidth. Spreading a job across racks can slow training substantially.

## Common mistakes

- Treating GPUs like CPU cores and allocating them one at a time.
- Preemption with no checkpoint contract.
- Measuring allocation instead of actual utilization.
- No plan for fragmentation.

## Go deeper

- Written for a general system design round, architecture first: [Design a GPU cluster scheduler](https://github.com/design-gurus/grokking-system-design/blob/main/questions/design-gpu-cluster-scheduler.md).
- The full worked solution: [Grokking the AI System Design Interview](https://www.designgurus.io/course/grokking-the-ai-system-design-interview).
