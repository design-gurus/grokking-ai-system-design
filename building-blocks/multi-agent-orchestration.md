# Multi-agent orchestration

> Splitting work across several agents, which is the answer interviewers hear too often and accept too rarely.

## What it is

Instead of one agent with every tool, you run several, each with a narrow role and its own tool set. Something coordinates them.

## The problem it solves

One agent with 40 tools picks the wrong one. One agent with a 12-step task loses the thread partway through. Splitting the work gives each agent a smaller decision and a smaller context.

That is the honest case for it, and it is narrower than most candidates suggest.

## The patterns

| Pattern | Shape | Fits |
|---------|-------|------|
| Supervisor | One agent routes to specialists and assembles the result | Distinct domains with distinct tools |
| Pipeline | Fixed sequence, each stage does one job | A known process, like draft then review then format |
| Parallel fan-out | Several agents work on independent parts at once | Tasks that genuinely do not depend on each other |
| Debate or critique | One produces, another checks | High-value output where a second opinion pays |

A pipeline is usually not multi-agent at all. It is a workflow with model calls in it, which is simpler to build, cheaper, and far easier to test. If a pattern can be written as ordinary code calling models at fixed points, write it that way.

## What it costs

Each of these is a real reason interviewers push back.

**Cost multiplies.** Every agent has its own system prompt, its own tool definitions, and its own accumulating history. Three agents on one task is more than three times one agent, because context is duplicated across all of them.

**Errors compound.** If each agent is right 90 percent of the time, three in sequence are right 73 percent of the time. Chains of agents are less reliable than their parts, not more.

**Handoffs lose information.** Agent A summarizes for agent B. What A left out, B cannot recover, and B has no way to know something is missing.

**Debugging gets much harder.** A wrong final answer has to be traced back through several trajectories to find where it went wrong.

**Latency adds up.** Sequential agents each wait for their own model calls.

## When it is genuinely right

- The sub-tasks need different tools, and the tool sets should not be combined for permission reasons. A billing agent and a shipping agent with separate access is a real design, not an aesthetic one.
- The sub-tasks are independent and can run at the same time, so the wall clock actually improves.
- One role must be isolated from another's context, for example a checker that should not see the writer's reasoning.

Outside those cases, prefer one agent with a small tool set and a bounded loop. See [the agent loop and its bounds](agent-loop-and-bounds.md).

## How to make it work if you do it

- Define the handoff as a schema, not as free text. Fixed fields survive a handoff; a paragraph does not.
- Give the whole task one shared budget, not one per agent. Otherwise the total cost is unbounded.
- Log every agent's trajectory with the same task id, so the run can be reconstructed.
- Evaluate each agent on its own, then the assembled system. An end-to-end score cannot tell you which one is weak.

## Trade-offs

| Pro | Con |
|-----|-----|
| Each agent has a smaller decision to make | Cost multiplies across agents |
| Tool sets can be separated by permission | Accuracy compounds downward through a chain |
| Independent parts can run in parallel | Handoffs lose information silently |
| Roles can be isolated for review or checking | Debugging spans several trajectories |

## Numbers to know

- Three sequential agents at 90 percent each give about 73 percent end to end.
- Duplicate system prompts and tool definitions are paid for in every agent, on every step.
- Parallel fan-out improves wall clock only when the parts are truly independent.

## How to talk about it in an interview

The signal is that you argue against it first.

**The sentence to say.** "I would start with one agent and a small tool set. I would split only if two sub-tasks need tool sets that should not share permissions, or if they are independent enough to run in parallel. Three agents in a chain at 90 percent each are 73 percent end to end, and cost multiplies because context is duplicated."

Expect these follow-ups:

- What the handoff between agents actually contains.
- How you would debug a wrong answer across several agents.
- How you evaluate the system. Per agent first, then end to end.

## Go deeper

- Related: [the agent loop and its bounds](agent-loop-and-bounds.md), [design a customer support agent](../questions/design-a-customer-support-agent.md)
- Course: [Grokking the AI System Design Interview](https://www.designgurus.io/course/grokking-the-ai-system-design-interview)
