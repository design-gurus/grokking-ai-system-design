# Design a prompt management service

> Treat prompts as versioned artifacts with a release process, because a one-line edit changes production behavior with no deploy and no test.

**Family:** Platform
**Also asked as:** design prompt versioning, design a prompt registry.

## 1. Frame the problem

Prompts start life in source code, which is fine. They end up edited by people who do not deploy code, tuned weekly, and tested by reading a few outputs.

The failure is specific. A small wording change is shipped, and quality drops for a subset of requests. Nobody can say which version was live when the complaints started.

The platform's job: store prompts as versioned artifacts, gate changes on evaluation, roll them out gradually, and make rollback instant.

Scale to assume: 20 teams, 500 prompts, several changes a day, 3,000 requests per second reading them.

## 2. Define success

| Metric type | Metric | Target |
|-------------|--------|--------|
| Safety | Share of prompt changes that ran evaluation before reaching production | Total |
| Speed | Time to roll back a bad prompt | Seconds |
| Service | Added latency to fetch the active prompt | Effectively zero, from cache |
| Traceability | Share of logged responses that record the exact prompt version used | Total |

The last one is the quiet requirement. Without a version stamped on every response, no incident can be traced and no experiment can be measured.

## 3. Map the data

- **Prompt.** A named artifact with a template, declared variables, the model it targets, and its sampling settings. Immutable per version.
- **Version.** Content, author, timestamp, parent version, and its evaluation results.
- **Deployment.** Which version is active in which environment, and for what share of traffic.
- **Usage log.** Every request records the prompt name and version, joined later to quality and cost.

Prompts are immutable once created. Editing produces a new version. That single rule is what makes rollback and attribution possible.

## 4. The core design choice

**Prompts are configuration with a release process, not content in a database.**

Three parts:

1. **A registry** holding versions. Immutable, append-only, with a diff view.
2. **A gate.** Promoting a version to production requires an evaluation run against the prompt's dataset, comparing it to the version currently live. See [design a model evaluation pipeline](design-a-model-evaluation-pipeline.md).
3. **A deployment record**, read by the serving path, that says which version is active and at what percentage.

**Templating needs to be strict.** Declare the variables a prompt takes, validate them at render time, and fail loudly on a missing one. A template that silently renders "Hello , your order" is a bug that reaches users.

Model settings belong with the prompt. A prompt tuned at one temperature, on one model, is not the same artifact when either changes, so the version pins both.

## 5. The serving path

```
application --> [SDK: get("summarizer", vars)]
                  --> local cache (refreshed every few seconds)
                  --> render and validate variables
                  --> call the model, stamping name and version
```

**Never a synchronous fetch per request.** The prompt service is not on the hot path. Clients hold a local cache with a short refresh, and they keep serving the last known good version if the service is unavailable.

Rollout is by percentage, keyed on a stable hash so a user does not switch versions mid-conversation. Rollback is setting the active version back, which propagates within one cache refresh.

## 6. Keep it working

- **Who can edit is a real question.** Non-engineers editing production prompts is the point of the service, and it is only safe because of the evaluation gate and instant rollback.
- **Watch quality per version**, not overall. The dashboard should show the metric split by active version during a rollout.
- **Prompts rot when the model changes.** A provider model upgrade can change behavior with no prompt edit, so pin model versions and re-run evaluation when they change.
- **Prune.** Most prompts stop being used. Track last-served time and archive the rest.

## 7. Count the costs

The service itself is cheap. Its cost is the evaluation runs it triggers, which is the price of the gate and worth paying.

The savings are real. Prompt length is a per-request cost, and a registry shows which prompts are long. You can then measure whether the length earns its tokens, and shorten them centrally.

## Common follow-ups

- **What happens when the service is down?** Clients serve from cache. A prompt fetch must never fail a user request.
- **How do you roll back?** Point the active version back. It propagates in one cache refresh.
- **How do you know a new prompt is better?** The evaluation gate first, then a percentage rollout with quality tracked per version.
- **Do you allow product managers to edit prompts?** Yes, and that is exactly why the gate and rollback exist.

## Common mistakes

- Mutable prompts, so history and attribution are lost.
- A synchronous prompt fetch on the request path.
- No version stamped on the response log.
- Treating prompts as text and ignoring the model and sampling settings they depend on.

## Go deeper

- Related: [model gateway and routing](../building-blocks/model-gateway-and-routing.md)
- The full worked solution: [Grokking the AI System Design Interview](https://www.designgurus.io/course/grokking-the-ai-system-design-interview)
