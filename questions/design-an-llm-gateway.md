# Design an LLM gateway

> One service in front of every model, so that cost, quotas, fallback, and logging exist once instead of in twenty applications.

**Family:** Platform
**Also asked as:** design an AI proxy, design a multi-provider LLM router.

## 1. Frame the problem

Every product team calls models. Without a shared entry point, three things are true. Nobody can answer who spent the money. No application has a fallback when a provider fails. No evaluation set can be built, because nothing was logged the same way.

The gateway's job: accept a model request, apply policy, send it to the right model, and return the response, adding only a few milliseconds.

Scale to assume: 20 teams, 3 external providers plus an internal fleet, 3,000 requests per second, most of them streaming.

## 2. Define success

| Metric type | Metric | Target |
|-------------|--------|--------|
| Service | Added latency, and added time to first token | Single-digit milliseconds |
| Reliability | Availability, and successful failover rate during a provider incident | Higher than any single provider |
| Cost | Spend per team and per feature, and savings from routing and caching | Attributable to the request |
| Adoption | Share of company model traffic through the gateway | Near total, or the data is incomplete |

Availability is the reason this exists. A gateway that fails over across providers is more available than any provider it calls, and that is the sentence to say.

## 3. Map the data

- **Request log.** Prompt, response, model, version, token counts, latency, cost, caller, and trace id. Sampled or full, depending on the privacy policy.
- **Configuration.** Routing rules, quotas, model aliases, prompt versions. Read on every request, so it lives in a fast store with a local cache.
- **Usage counters.** Tokens per team per window, for quotas and billing.
- **Credentials.** Provider keys held by the gateway, never by applications. This is half the security value of the design.

Applications call a model alias, for example "summarizer-v2", not a provider model name. That indirection is what lets you change models without a deploy in twenty repositories.

## 4. The core design choice

**Stateless workers plus a fast configuration store.** Every instance can serve any request, so scaling is horizontal and an instance failure costs nothing. Configuration is cached locally with a short refresh, so a config store outage does not take the gateway down.

**Quota enforcement is the one piece of shared state.** A distributed counter per tenant per window, with a small local allowance to avoid a store round trip on every request. Approximate counting is acceptable here. Exactness is not worth the latency.

**Routing** by task, by a cheap difficulty classifier, or by cascade. See [model gateway and routing](../building-blocks/model-gateway-and-routing.md) for how to choose.

## 5. The serving path

```
[request] --> [auth, resolve alias, check quota]        2 ms
          --> [cache lookup: exact, then semantic]      20 ms on a hit
          --> [route to provider or internal fleet]
          --> [stream tokens straight back to the caller]
          --> [async: log, count tokens, record cost]
```

**Streaming shapes the whole design.** The gateway cannot buffer the response to inspect it, or it adds seconds to the first token. So logging, token counting, and cost recording happen as the stream passes and are finalized after it ends. Any output filter has to work on chunks, or be accepted as a latency cost. See [guardrails and safety filters](../building-blocks/guardrails-and-safety-filters.md).

**Retries need care.** A timeout on a streaming request that already emitted tokens cannot be retried transparently. Retry only before the first token, and make write-like operations idempotent.

## 6. Keep it working

- **Test the fallback path on a schedule.** Prompts are not portable across providers, and tool-calling formats differ. An untested fallback fails during the outage it exists for.
- **Circuit-break per provider.** When one starts failing, stop sending to it for a period rather than timing out every request.
- **Watch cost per team daily.** A prompt change that triples token use should raise an alert, not a surprise invoice.
- **Version prompts and models together**, and roll them out like code, with a small share of traffic first.

## 7. Count the costs

The gateway itself is cheap. It exists to control a much larger bill.

Report the savings it produces: cache hit rate, share of traffic routed to smaller models, and the spend avoided. That is how a platform team justifies the service.

Run it in at least two zones. It is now on the critical path of every AI feature in the company.

## Common follow-ups

- **What happens when the gateway is down?** Multiple instances, health-checked, plus a documented direct path for the most critical caller.
- **How do you count tokens for a streaming response?** As chunks pass, finalized at the end, with the count reconciled against the provider's reported usage.
- **How do you stop one team exhausting a shared provider limit?** Per-tenant token quotas enforced before the call.
- **Do you cache?** Exact caching always. [Semantic caching](../building-blocks/semantic-caching.md) with the tenant and permission scope in the key.

## Common mistakes

- Buffering streams, which destroys time to first token.
- Retrying a partially streamed response.
- Quota counting that requires a synchronous shared store read per request.
- No logging plan, so the company still cannot build an evaluation set.

## Go deeper

- Written for a general system design round, architecture first: [Design an LLM gateway](https://github.com/design-gurus/grokking-system-design/blob/main/questions/design-llm-gateway.md).
- The full worked solution: [Grokking the AI System Design Interview](https://www.designgurus.io/course/grokking-the-ai-system-design-interview).
