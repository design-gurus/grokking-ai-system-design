# Design a customer support agent

> An agent that reads a customer's message, looks up their account, takes a real action like issuing a refund, and knows when to hand the conversation to a person.

**Family:** Agentic
**Also asked as:** design an AI support system, design an autonomous ticket resolver.

## 1. Frame the problem

An agent is a loop. The model receives a goal, chooses a tool, reads the result, and chooses again, until it decides it is done. That loop is the whole design, and every hard part of this question is a bound on it.

The task: resolve common support requests end to end, and escalate the rest to a human with the context already gathered.

Scale to assume: a food delivery company with 200,000 tickets a day. About 60 percent come from four request types: where is my order, refund a missing item, change an address, and cancel.

The first design decision is not a model choice. **Most of these requests do not need an agent.** "Where is my order" is a lookup with a template answer. Say that out loud, then scope the agent to the requests that genuinely need several steps and a judgment call.

## 2. Define success

| Metric type | Metric | Target |
|-------------|--------|--------|
| Offline | Task success on a golden set of recorded tickets | Stated per request type |
| Offline | Trajectory quality: right tools, right order, no wasted calls | Reviewed on a sample |
| Online | Resolution rate without human help, repeat contact rate within 7 days | Resolution up, repeat contacts flat |
| Guardrail | Wrong refunds issued, escalation latency, cost per resolved ticket | Wrong refunds near zero |

Repeat contact rate exists to catch one failure. An agent can close a ticket by saying something agreeable and solving nothing. Resolution rate goes up, and the customer writes again on Tuesday. Measure both together.

Cost per resolved ticket is the number the business cares about. Compare it against the cost of a human agent handling the same ticket.

## 3. Map the data

**Tools are the data layer.** An agent's capability is exactly the set of tools it has, so list them with their permissions:

| Tool | Read or write | Bound |
|------|---------------|-------|
| Look up order | Read | Only orders belonging to the authenticated customer |
| Look up policy | Read | Retrieval over the support knowledge base |
| Issue refund | Write | Hard cap on amount, one per order, logged |
| Update address | Write | Only before dispatch |
| Escalate to human | Write | Always available |

**Authorization belongs in the tool, not in the prompt.** The model must never be the thing that decides whether a customer may read an order. Every tool checks the caller's identity itself. A prompt instruction is a suggestion. A permission check is a control.

**Labels** come from resolved tickets and from human review. A golden set of a few hundred real tickets is what every evaluation run scores against. Each entry holds the correct final outcome and an acceptable tool sequence. Build it before building the agent.

**Memory** has three tiers: the current conversation, facts about this customer from the account system, and summaries of past contacts. Do not put everything in the prompt. Retrieve what the current step needs.

## 4. Pick the model

Start with the simplest control structure that solves the request type, and only add freedom where it is needed.

| Structure | When it is right |
|-----------|------------------|
| Fixed workflow, no model deciding order | Single-step requests like order status |
| Model chooses from a small tool set, one step at a time | Most support requests |
| Planning agent that writes a multi-step plan | Rare, and hard to evaluate |
| Several specialized agents | Almost never in support, and it multiplies failure modes |

A large model handles the reasoning and tool choice. A small model can handle classification and the final message. Routing by request type saves real money.

**Say the control ladder out loud.** Interviewers are watching for whether you pick the most autonomous option by default. The senior answer picks the least autonomy that does the job.

## 5. The serving path

Budget: the customer sees a reply in a few seconds, and the whole resolution finishes in under a minute.

```
[message] --> [classify request type]
          --> [fixed workflow]            for simple types
          --> [agent loop]                for the rest
                 |  choose tool -> call tool -> read result
                 |  repeat, bounded
                 --> [answer]  or  [escalate with context]
```

Every loop needs four bounds, and an interviewer will ask for them by name:

1. **Step limit.** Stop after N tool calls, usually 5 to 10.
2. **Time limit.** Stop after a wall clock budget.
3. **Cost limit.** Stop after a token budget.
4. **Repeat detection.** Stop when the same tool is called with the same arguments twice.

When a bound is hit, the agent escalates. It never fails silently and it never tries again forever.

**Write actions need approval rules.** A refund under a threshold is automatic. Above it, the agent prepares the action and a human approves. That threshold is a product decision you can raise as confidence grows, which is a good thing to offer.

**Idempotency matters more here than in most systems.** A retried refund tool call must not issue two refunds. Every write tool takes an idempotency key derived from the ticket and the action.

## 6. Keep it working

- **Log the full trajectory.** Every prompt, tool call, argument, result, and the final message. Without this you cannot debug a bad refund, and you cannot build the golden set.
- **Evaluate trajectories, not just outcomes.** An agent that reaches the right answer after six pointless tool calls is expensive and fragile. See [evals/](../evals/).
- **Sample and review.** A share of resolved tickets goes to human review every day, weighted toward write actions.
- **Watch for policy drift.** When the refund policy changes, the knowledge base changes, and the agent's behavior changes with no deploy. That is a feature and a risk, so policy edits should trigger the evaluation suite.
- **Shadow mode first.** Run the agent alongside human agents without sending its messages, and compare its proposed actions to theirs. This is the cheapest way to build confidence before going live.

## 7. Count the costs

Per resolved ticket: perhaps 6 model calls, 3,000 input tokens each as the conversation and tool results accumulate, and short outputs. That is roughly 20,000 tokens per ticket. Multiply by the share of 200,000 daily tickets you route to the agent.

The accumulating context is the cost driver. Each step re-sends the history, so cost grows faster than linearly with the number of steps. That is the strongest argument for the step limit, and for summarizing tool results instead of pasting them whole.

Levers: cap steps, summarize tool output, and use a small model for the final message. The largest one is routing simple types to a fixed workflow, with no agent at all.

## Common follow-ups

- **What if the agent hallucinates a policy?** Policies come from retrieval, and the answer cites them. An uncited policy claim is a bug the evaluation suite tests for.
- **How do you hand off to a human?** With a summary, the trajectory, and the customer's original words. The worst handoff makes the customer repeat themselves.
- **What stops a customer from talking the agent into a refund?** The refund tool's own limits, not the model's judgment. Prompt injection is expected, so the limit must live in the tool, not in the model.
- **How do you roll out a prompt change?** Like a model change: run the evaluation suite, then ship to a small share of traffic and watch repeat contact rate.

## Common mistakes

- No bounds on the loop.
- Permission checks written into the prompt instead of the tools.
- Multi-agent designs offered before a single agent has been shown to fail.
- Measuring resolution rate alone, with no repeat contact metric.
- No plan for evaluating the sequence of steps, only the final answer.

## Go deeper

The full worked solution, including the agentic capstone that runs on this same scenario: [Grokking the AI System Design Interview](https://www.designgurus.io/course/grokking-the-ai-system-design-interview).
