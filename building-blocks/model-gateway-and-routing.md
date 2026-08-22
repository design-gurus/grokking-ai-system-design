# Model gateway and routing

> One service every model call goes through, so that quotas, cost, fallback, and logging exist in one place instead of twenty.

## What it is

A model gateway sits between your applications and the models, whether those models are hosted by a provider or served by you. Every request goes through it.

It is the same idea as an API gateway, with three additions that only matter for models. Cost is per request and large. Providers fail in ways you must survive. A cheaper model can often do the job.

## The problem it solves

Without a gateway, each team calls the provider directly with its own key. Four things follow, and all four show up as incidents:

- Nobody can say which team spent the money, so nobody can cut it.
- One team's retry loop exhausts the shared rate limit and takes down another team's feature.
- A provider outage becomes an outage in every product, because no application has a fallback.
- Prompts and responses are logged inconsistently, so evaluation and debugging have no data.

## What it does

| Function | Why it belongs here |
|----------|---------------------|
| Authentication and quotas | Per-team limits, so one caller cannot exhaust the pool |
| Cost attribution | Tokens counted per team, feature, and user |
| Routing | Send each request to the model that fits it |
| Fallback | A second provider or a smaller model when the first fails |
| Retries and timeouts | One correct implementation instead of twenty guesses |
| Caching | Exact and [semantic caching](semantic-caching.md) in one place |
| Logging | Prompt, response, model version, latency, and cost for every call |
| Safety hooks | Input and output filters applied uniformly. See [guardrails](guardrails-and-safety-filters.md) |

The logging is the part teams regret skipping. Without stored prompts and responses you cannot build an evaluation set, debug a complaint, or measure a model change.

## Routing

```
request --> [classify difficulty] --> small model   (most traffic)
                                  --> large model   (hard cases)
                                  --> specialist    (code, long context)
```

Routing is where the largest savings are, because most traffic is easy. Three ways to decide, in increasing sophistication:

1. **By task.** The application declares what it is doing, and the gateway maps that to a model. Simple and predictable.
2. **By a cheap classifier.** A small model estimates difficulty and picks. More savings, and it needs its own evaluation.
3. **By cascade.** Try the small model, check the answer with a cheap verifier, escalate on failure. Best savings when verification is reliable, and it adds latency on the escalated share.

Every routing rule is a quality decision. A wrong route gives a worse answer, so the routing accuracy has to be measured like any other model.

## Fallback needs a matching prompt

A fallback to another provider only works if the prompt still behaves there. Prompts are not portable, and neither are tool-calling formats or structured output modes.

So the fallback path must be tested regularly, not written once and trusted. Say that out loud: an untested fallback is a fallback that fails during the outage it exists for.

## Trade-offs

| Pro | Con |
|-----|-----|
| One place for cost, quotas, and logs | One more hop, adding a few milliseconds |
| Provider changes do not touch application code | A shared service is a shared failure point |
| Routing cuts spend without touching products | Every routing rule can pick wrong |
| Prompt and model versions are managed centrally | Teams may want provider features the gateway does not expose |

## Numbers to know

- Gateway overhead should stay in single-digit milliseconds, next to model calls of hundreds of milliseconds.
- Routing easy traffic to a smaller model commonly saves a large multiple on those requests. Size it from your own traffic mix.
- Streaming must pass through untouched, or the gateway adds latency to the first token, which is the number users feel.
- Run it as at least two instances. It is now on the critical path of every product.

## How to talk about it in an interview

**The sentence to say.** "All model calls go through a gateway, so quotas, cost per team, retries, caching, and logging exist once. It routes by task, sending most traffic to a small model. It fails over to a second provider, on a prompt that is tested on both."

Expect these follow-ups:

- How you would attribute cost to a feature. Tokens counted at the gateway, tagged by caller.
- What happens when the gateway itself is down. Multiple instances, and a documented direct path for the most critical caller.
- How you decide which requests can use the smaller model.

## Go deeper

- Cost math: [numbers you should know](../cheat-sheets/ai-numbers.md)
- Course: [Grokking the AI System Design Interview](https://www.designgurus.io/course/grokking-the-ai-system-design-interview)
