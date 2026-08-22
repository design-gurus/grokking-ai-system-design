# The agent loop and its bounds

> An agent is a while loop with a model inside it. Every hard part of agent design is a limit on that loop.

## What it is

The loop is short:

```
while not done:
    model decides: answer, or call a tool
    if tool: run it, append the result, continue
```

That is the entire mechanism. What separates a demo from a production system is not the loop. It is the set of limits around it.

## The problem it solves

A single model call cannot gather information, act, and check the result. The loop lets the model take a step, see what happened, and choose the next one.

The cost of that freedom is that nothing stops it. Without limits, a confused agent calls the same tool 40 times, spends real money, and never returns. There is no exception thrown, because nothing has failed.

## The four bounds

Name all four in an interview. Candidates usually name one.

| Bound | What it stops | Typical value |
|-------|---------------|---------------|
| Step limit | Endless tool calling | 5 to 10 steps |
| Time limit | A slow tool holding the request open | Tens of seconds |
| Cost limit | Token spend on one task | A stated budget per task |
| Repeat detection | The same call with the same arguments, twice | Immediate stop |

Repeat detection is the one people forget, and it catches the most common real failure. An agent that gets an unhelpful tool result often tries the identical call again, because the model's view of the world has not changed.

**Hitting a bound is not an error.** It is a defined outcome: stop, summarize what was learned, and escalate to a person or return a partial answer. An agent that fails silently at a limit is worse than one with no limit, because nobody finds out.

## Cost grows faster than the step count

Each step re-sends the conversation and every tool result so far. Step 6 carries everything from steps 1 to 5.

So doubling the step limit more than doubles the cost. Two consequences worth saying:

- Summarize tool results before appending them, rather than pasting whole payloads.
- Keep the step limit as low as the task allows, and measure how many steps real tasks actually use.

## What to log

The trajectory is the unit of debugging and the unit of evaluation. Log every step: the prompt, the chosen tool, the arguments, the result, the latency, and the token count.

Without this you cannot explain a bad action after the fact, and you cannot build the golden set that evaluation needs. See [evals](../evals/).

## Use less autonomy where you can

The loop is not always the right shape. Ordered by how much freedom the model gets:

1. **A fixed workflow.** Code calls the model at defined points. Predictable, testable, and correct for most requests.
2. **A bounded loop with a small tool set.** The model picks the next step. The common middle ground.
3. **A planning agent.** The model writes a multi-step plan first. Harder to evaluate, and it fails in ways that are hard to reproduce.

Pick the least autonomy that solves the task. Interviewers watch for candidates who start at the top of that list.

## Trade-offs

| Pro | Con |
|-----|-----|
| Handles tasks whose steps are not known in advance | Non-deterministic, so testing is statistical |
| Recovers from a failed step by trying another route | Cost and latency vary per request |
| One agent covers many request types | A wrong early step sends the whole trajectory wrong |
| Bounds make the worst case predictable | Bounds that are too tight cause escalations that a person must handle |

## Numbers to know

- Most real tasks that succeed finish in 3 to 6 steps. Measure yours before setting the limit.
- Each step is a full model round trip, so 6 steps is several seconds even with fast tools.
- Token cost per task grows roughly with the square of the step count, because history accumulates.

## How to talk about it in an interview

**The sentence to say.** "The loop is bounded four ways: a step limit, a wall clock limit, a token budget, and repeat detection. When any bound is hit the agent escalates with a summary, so a stuck task becomes a human handoff rather than a silent failure."

Expect these follow-ups:

- What happens when the agent hits the limit mid-task.
- Why cost is not linear in the number of steps.
- Whether this task needs an agent at all.

## Go deeper

- Related: [tool calling](tool-calling.md), [agent memory](agent-memory.md), [design a customer support agent](../questions/design-a-customer-support-agent.md)
- Course: [Grokking the AI System Design Interview](https://www.designgurus.io/course/grokking-the-ai-system-design-interview)
