# Quantization and distillation

> Two ways to make a model smaller. One compresses the weights you have. The other trains a new, smaller model to copy the old one.

## What it is

**Quantization** stores each weight in fewer bits. A model held in 16-bit numbers can often be held in 8 or 4 bits instead, with a small quality cost.

**Distillation** trains a small model, the student, on the outputs of a large model, the teacher. The student learns the teacher's behavior on the task you care about, and it can be far smaller than any compressed version of the teacher.

## The problem it solves

Model size sets three costs at once: memory, which caps concurrency, memory bandwidth, which caps decode speed, and hardware, which caps the bill. A model that does not fit on one GPU has to be split across several, and the systems problem gets harder immediately.

Making the model smaller improves all three at the same time. The only question is what it costs in quality.

## Quantization: the memory table

At 2 bytes per parameter, memory in gigabytes is roughly twice the parameter count in billions.

| Precision | Bytes per parameter | An 8B model | A 70B model |
|-----------|--------------------|-------------|-------------|
| 16-bit | 2 | About 16 GB | About 140 GB |
| 8-bit | 1 | About 8 GB | About 70 GB |
| 4-bit | 0.5 | About 4 GB | About 35 GB |

The practical effect is not a percentage saving. It is a threshold. 8-bit is what puts a 70B model on a single 80 GB GPU, which removes the whole problem of splitting a model across machines.

Quality loss at 8-bit is usually small. At 4-bit it is visible on harder tasks. The honest answer in an interview is that you measure it on your own evaluation set, rather than trusting a general claim.

## Distillation: a different trade

Quantization keeps the same model and shrinks its numbers. Distillation replaces the model.

The student is trained on inputs plus the teacher's outputs, often including the teacher's full probability distribution rather than just the final answer. That extra signal is why distillation works better than training the small model on the raw labels.

The win is largest when the task is narrow. A general-purpose large model does thousands of things. Your product needs one of them. A small student trained on that one thing can match the teacher on it, at a fraction of the cost.

## Which one to pick

| Situation | Use |
|-----------|-----|
| Model nearly fits the hardware you have | Quantization |
| Serving cost is high and the task is narrow | Distillation |
| You have no labeled data but do have a strong model | Distillation, using the teacher to label |
| You cannot change model behavior at all | Quantization, measured carefully |
| Latency is the problem and the model is already small | Neither. See [speculative decoding](speculative-decoding.md) |

They combine. A distilled student can also be quantized.

## Trade-offs

| Pro | Con |
|-----|-----|
| Quantization is fast to apply and needs no training | Quality loss is task-dependent and must be measured |
| Quantization can move a model onto one GPU | Very low precision degrades reasoning-heavy tasks first |
| A distilled student can be many times cheaper to serve | Distillation is a training project with data requirements |
| A student can be specialized to your traffic | The student inherits the teacher's mistakes, and stops improving when the teacher does |

## Numbers to know

- 8-bit halves memory and usually costs little quality. 4-bit halves it again with a larger, task-dependent cost.
- Smaller weights also mean less memory read per token, so decode speeds up along with the memory saving.
- A distilled student is often 10 to 20 times smaller than its teacher for a single task.
- Distillation data: thousands to hundreds of thousands of teacher-labeled examples, depending on task breadth.

## How to talk about it in an interview

The signal is that you name the quality measurement, not just the saving.

**The sentence to say.** "I would quantize to 8-bit first, because it puts the model on one GPU and removes the sharding problem. I would check the quality drop on our own evaluation set before shipping it. If serving cost is still the constraint, the task is narrow enough to distill a much smaller student."

Expect these follow-ups:

- How you decide the quality loss is acceptable. A fixed evaluation set and a stated threshold, not a general claim.
- Which tasks degrade first under aggressive quantization. Long reasoning chains and rare knowledge.
- Whether you would distill from a hosted model, and what the licence says about that.

## Go deeper

- Memory math: [numbers you should know](../cheat-sheets/ai-numbers.md)
- Course: [Grokking the AI System Design Interview](https://www.designgurus.io/course/grokking-the-ai-system-design-interview)
