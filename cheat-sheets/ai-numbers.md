# Numbers you should know

> The anchors for back-of-the-envelope math in an AI design question. Round numbers you remember beat precise numbers you have to look up.

Interviewers do not check your arithmetic. They check whether you know which quantity matters. Five anchors cover almost every estimate in this round.

## The five anchors

| Anchor | Value to remember | Why it matters |
|--------|-------------------|----------------|
| Tokens per word | About 1.3 tokens per English word | Turns any page of text into a token count |
| Page of text | About 500 words, so about 650 tokens | The unit for prompts, documents, and chunks |
| Model memory | 2 bytes per parameter at half precision | An 8 billion parameter model needs about 16 GB just to load |
| GPU memory | 80 GB on a high-end data center GPU | Tells you how many models fit, and how much is left for the KV cache |
| Embedding size | 768 to 1536 floats, so 3 to 6 KB per vector | Turns a corpus size into an index size |

## Latency anchors

| Operation | Rough time |
|-----------|-----------|
| Vector search over millions of items | 5 to 50 ms |
| Feature fetch from an online store | 1 to 10 ms |
| A small ranking model scoring 500 candidates | 10 to 50 ms |
| Time to first token from a hosted LLM | 200 ms to 1 second |
| Generating 200 tokens at 50 tokens per second | About 4 seconds |
| Cross-encoder reranking of 50 candidates | 50 to 200 ms |

The last two matter most. Generation is slow enough that you cannot hide it, so you stream. The rule that falls out: **budget for time to first token, not for total time.** A user tolerates four seconds of streaming and does not tolerate four seconds of a blank screen.

## Token cost math

Prices change constantly, so memorize the shape of the calculation, not the price.

```
cost per day = requests per day
             x (input tokens + output tokens x output multiplier)
             x price per token
```

Output tokens usually cost several times more than input tokens. Say that out loud, because candidates who quote one blended price are guessing.

Worked example, using one dollar per million input tokens as a placeholder:

- 1 million requests per day
- 800 input tokens, 200 output tokens per request
- Input: 800 million tokens per day
- Output: 200 million tokens per day, priced higher

That is roughly a five-figure monthly bill for one feature. The point of saying it out loud is the next sentence: **which lever do you pull?**

| Lever | Typical saving | Cost |
|-------|---------------|------|
| Route easy requests to a smaller model | 5 to 20 times on those requests | A routing mistake gives a worse answer |
| Cache repeated or near-repeated questions | 30 to 80 percent on a support workload | Stale answers, cache invalidation |
| Shorten the prompt, or cache the fixed part | 20 to 50 percent of input tokens | Engineering time |
| Cap output length | Direct, linear | Truncated answers |

## GPU memory math

To serve a model yourself, memory is the first constraint, not speed.

```
weights = parameters x bytes per parameter
KV cache = 2 x layers x heads x head dim x bytes x sequence length x batch size
```

Two rough facts carry most conversations. An 8 billion parameter model at half precision needs about 16 GB for weights. A 70 billion parameter model needs about 140 GB. That does not fit on one 80 GB GPU, so it must be split across two or more. That split is why serving large models is a systems problem.

The KV cache holds the attention state for every token in every active request, so it grows with concurrency and with conversation length. It is often the reason a server runs out of memory long before it runs out of compute. See [continuous batching and the KV cache](../building-blocks/continuous-batching-and-kv-cache.md).

## Corpus and index math

```
vectors = documents x chunks per document
index size = vectors x dimensions x 4 bytes
```

One million documents at 10 chunks each is 10 million vectors. At 768 dimensions and 4 bytes per float, that is about 30 GB of raw vectors. A graph index adds roughly 20 to 50 percent on top. This fits in memory on one large machine, which is a useful thing to notice out loud before proposing a distributed vector database.

## Training anchors

You will rarely be asked to size a training run, but two numbers help.

- A recommendation or ranking model retrains in hours, on tens of machines, and often runs daily.
- Fine-tuning a small language model on a few thousand examples takes hours on a small number of GPUs. Pretraining a base model is a different budget entirely, measured in thousands of GPU-months, and is out of scope in an interview.

## How to use these

Do the estimate at step 7 of the [framework](ai-system-design-framework.md), not at the start. Round hard, say the assumption before the number, and finish with the lever you would pull. An estimate with no decision attached to it is wasted time.

## Go deeper

The full lesson on back-of-the-envelope math for AI systems, with worked GPU fleet and token bill examples: [Grokking the AI System Design Interview](https://www.designgurus.io/course/grokking-the-ai-system-design-interview).
