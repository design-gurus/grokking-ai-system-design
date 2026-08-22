# Agent memory

> Deciding what an agent carries with it, given a context window that is a budget rather than a storage system.

## What it is

Memory is everything the agent knows at the current step that did not arrive in the current message. Three tiers, with different lifetimes and different costs.

| Tier | Holds | Lifetime |
|------|-------|----------|
| Working | The current conversation and recent tool results | This task |
| Long-term facts | Account data, preferences, past decisions | Permanent, and it lives in your database |
| Summaries | Compressed history of earlier turns or past sessions | Rolling |

## The problem it solves

The context window is finite, and every token in it is paid for on every step. An agent that appends everything runs out of room mid-task, and it pays more for each step than the one before.

The opposite failure is just as visible to users. An agent that forgets what the customer said two messages ago makes them repeat themselves, which is the single most common complaint about support systems.

## How it works

The rule is simple to state and easy to get wrong: **retrieve what this step needs, do not carry everything.**

```
step context = system prompt
             + a summary of earlier turns
             + the last few turns in full
             + facts retrieved for this step
             + the current tool result
```

Long-term memory is not a bigger prompt. It is your existing systems, queried like any other data:

- Account facts come from the account database, by id. Not from a vector search, and not from a summary.
- Documents and past tickets come from retrieval, scoped to this user's permissions.
- Preferences learned across sessions live in a normal table with a schema you can inspect and edit.

Store structured facts as structured data. A preference stored as free text inside a summary cannot be corrected, audited, or deleted on request.

## Compaction

When the conversation grows past its budget, replace old turns with a summary. Two rules keep this from losing the thread:

- Keep the most recent turns in full. Recent detail matters most and summarization damages it most.
- Summarize toward the task. "The customer wants a refund on order A-4471, already confirmed the item was missing" is useful. A neutral recap of the conversation is not.

Summaries compound errors. A summary of a summary drifts, and the drift is invisible until the agent acts on something that was never said. Keep the raw log, and regenerate the summary from it when you can, rather than summarizing the summary.

## What memory writes cost you

An agent that writes to memory is a system that can teach itself something false. A misheard preference gets stored, then retrieved and repeated forever, and the user experiences it as the product being confidently wrong about them.

Three defenses. Write only from confirmed facts, never from inferences. Attach a source and a timestamp to every stored fact. Make stored memory inspectable and deletable by the user.

## Trade-offs

| Pro | Con |
|-----|-----|
| Continuity across turns and sessions | Every carried token is paid for on every step |
| Retrieved facts keep the prompt small | A retrieval miss looks like forgetting |
| Structured facts can be audited and corrected | Structure needs a schema, so it is real engineering |
| Summaries hold long conversations in budget | Summaries lose detail and can drift |

## Numbers to know

- Prompt cost grows with the number of steps, because history is re-sent each time. See [the agent loop](agent-loop-and-bounds.md).
- Keeping the last 3 to 5 turns in full, with a summary before them, covers most conversations.
- Retrieved facts should be a handful of short items, not pages. Measure whether the later ones change the answer.
- Personal data in memory is subject to deletion requests, so it needs a real store rather than a prompt.

## How to talk about it in an interview

**The sentence to say.** "Working memory is the last few turns in full plus a task-focused summary of the rest. Account facts are retrieved from the database by id, at the step that needs them, rather than carried in the prompt. Anything learned about the user is stored as structured, deletable data."

Expect these follow-ups:

- What happens when the conversation exceeds the context window.
- How you stop a wrong fact persisting forever.
- Where personal data lives, and how a deletion request is handled.

## Go deeper

- Related: [the agent loop and its bounds](agent-loop-and-bounds.md), [chunking strategies](chunking-strategies.md)
- Course: [Grokking the AI System Design Interview](https://www.designgurus.io/course/grokking-the-ai-system-design-interview)
