# PagedAttention and vLLM

> The design that showed LLM serving throughput is limited by memory management, and borrowed virtual memory paging to fix it.

## What it is

vLLM is an open-source serving system for large language models. PagedAttention is its central idea. Store the attention cache in small fixed-size blocks, the way an operating system stores process memory in pages. Earlier systems used one contiguous reservation per request.

## The problem it solves

Generating text requires keeping the key and value tensors for every token produced so far. That is the KV cache, and it is the memory that lets the model produce the next token without reprocessing the whole sequence.

Earlier serving systems reserved one contiguous block of memory per request, sized for the maximum possible output length. Two kinds of waste followed:

- **Internal waste.** A request that could have generated 2,000 tokens but stops at 100 still holds the full reservation.
- **External waste.** Free memory exists but is split into pieces too small to hold the next request's reservation.

Measurements in the vLLM paper put usable cache memory as low as 20 to 40 percent under this scheme. Since the number of requests you can run at once is set by cache memory, most of the hardware sat unused.

## Key design ideas

| Idea | How it works |
|------|--------------|
| Paged blocks | The cache is split into fixed-size blocks holding a few tokens each. A request gets blocks as it generates, and they need not be next to each other. |
| A block table | Each request keeps a table mapping its logical token positions to physical blocks, exactly like a page table. |
| Sharing by reference | Requests with the same prefix, for example one long system prompt, point at the same blocks instead of each holding a copy. |
| Copy on write | When two requests share a block and one needs to diverge, only that block is copied. |
| Continuous batching | The scheduler admits and retires requests every model step, so a finished request frees its blocks immediately. |

The result is that memory waste drops to roughly one partly filled block per request, and the freed memory turns directly into more concurrent requests. The paper reports two to four times the throughput of prior systems at the same latency.

## Trade-offs

- Attention has to be written to read from scattered blocks, so the kernel is more complex than a contiguous one.
- There is a small bookkeeping cost per step for the block tables.
- Sharing only helps when prefixes match exactly, so prompt design affects how much you save.
- Under overload the system now queues rather than failing on allocation, so admission control and a latency limit become your responsibility.

## What to say about it in an interview

The signal is that you know serving capacity is a memory question.

**The sentence to say.** "Throughput here is set by how many requests fit in the KV cache, not by raw compute. A paged cache removes the reservation waste. A shared system prompt is stored once instead of once per request. The same GPU then serves several times more concurrent users."

Expect the follow-up: what happens when memory runs out mid-generation. The answer is that the scheduler preempts a request. It either recomputes that cache later or swaps the blocks out. Both cost throughput, and both avoid a failure.

## Go deeper

- Building block: [continuous batching and the KV cache](../building-blocks/continuous-batching-and-kv-cache.md)
- Numbers: [GPU memory math](../cheat-sheets/ai-numbers.md)
- Course: [Grokking the AI System Design Interview](https://www.designgurus.io/course/grokking-the-ai-system-design-interview)
