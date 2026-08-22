# Semantic caching

> Reusing a stored answer for a question that is similar but not identical, and the reason that is riskier than a normal cache.

## What it is

An exact cache keys on the request text. If the same string arrives twice, the stored answer is returned.

A semantic cache keys on meaning. It embeds the question, searches for a stored question whose vector is close enough, and returns that answer.

## The problem it solves

Users ask the same thing in different words. "How do I cancel" and "how can I cancel my subscription" are one question and two cache keys. In a support workload, a large share of daily volume is a handful of questions phrased hundreds of ways.

An exact cache misses all of that. Each miss costs a full generation, which is the expensive part of the request.

## How it works

```
question --> embed --> nearest stored question
                       |
                       similarity above threshold? --> return stored answer
                       otherwise                    --> generate, then store
```

The threshold is the whole design. Set it high and the cache rarely hits. Set it low and the system confidently answers a question the user did not ask.

That is what makes this different from an ordinary cache. A normal cache miss costs latency. A semantic cache false hit returns a wrong answer that looks completely legitimate, with no error anywhere in the system.

## The rules that make it safe

**Scope the key by everything that changes the answer.** The user's tenant, their permissions, their language, and the model version all belong in the key. Two users with different document access must never share a cache entry. This is the most likely way a cache becomes a data leak.

**Do not cache personalized answers.** "What is my current balance" is a question whose answer depends on the asker. Cache the general policy answers, not the account-specific ones. A simple classifier or a rule on the question type is enough to decide.

**Set a short expiry and invalidate on content change.** When the underlying document changes, entries that came from it are stale. Storing the source ids with each entry makes that invalidation possible.

**Log hits and sample them for review.** A false hit rate you never measure is a false hit rate you do not know.

## Prompt caching is a different thing

Providers and serving runtimes also cache the processed form of a repeated prompt prefix, for example a long system prompt or a shared document. That is a compute optimization inside the model server, it is exact rather than semantic, and it never changes the answer. See [continuous batching and the KV cache](continuous-batching-and-kv-cache.md).

Both are worth doing. Only one of them can be wrong.

## Trade-offs

| Pro | Con |
|-----|-----|
| Removes generation cost on repeated questions | A false hit answers the wrong question convincingly |
| Answers return in milliseconds instead of seconds | Threshold tuning needs labeled pairs to get right |
| Reduces load during a traffic spike | Cache keys must carry permissions, or it leaks |
| Cheap to add on top of an existing pipeline | Stale entries survive content updates unless invalidated |

## Numbers to know

- Cache lookup: an embedding call plus a vector search, so roughly 20 to 50 ms, against seconds for generation.
- Hit rates on support and documentation workloads are commonly reported in the tens of percent. Measure your own before promising a saving.
- Expiry is usually minutes to hours, shorter than a normal cache because the source content moves.

## How to talk about it in an interview

**The sentence to say.** "I would cache generic answers by question embedding, with the tenant and permission scope in the key, and a threshold tuned on labeled question pairs. Account-specific questions are never cached, and a false hit is the risk I would monitor, because it returns a confident wrong answer with no error."

Expect these follow-ups:

- How you pick the similarity threshold, and how you measure a false hit.
- Why the cache key needs the permission scope.
- What invalidates an entry when a document is edited.

## Go deeper

- Cost levers: [numbers you should know](../cheat-sheets/ai-numbers.md)
- Course: [Grokking the AI System Design Interview](https://www.designgurus.io/course/grokking-the-ai-system-design-interview)
