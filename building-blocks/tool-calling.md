# Tool calling

> Letting a model do something instead of only saying something, and keeping the permission check somewhere the model cannot reach.

## What it is

A tool is a function the model can invoke: look up an order, search a corpus, send an email. You describe the tool's name, purpose, and arguments. The model responds with a structured call rather than prose.

Your code runs the function. The result goes back to the model, which then answers or calls another tool.

## The problem it solves

A language model has no access to current data and cannot change anything. Tool calling connects it to your systems, which is what turns a chat feature into a product that resolves a request.

It also fixes a reliability problem. Asking a model to output JSON in prose form produces JSON that is occasionally malformed. A tool call is a structured, validated response, so the parsing failure class mostly goes away.

## How it works

```
user message + tool definitions --> model
model --> {tool: "lookup_order", args: {order_id: "A-4471"}}
your code --> validate --> check permissions --> run --> result
result --> model --> answer, or the next tool call
```

Every step between the model's request and your database is yours. That gap is where correctness and security live.

## The rules that matter

**Authorization belongs in the tool.** The tool receives the caller's identity from the session, not from the model's arguments. It checks permissions itself, every time.

A prompt that says "only look up orders belonging to this customer" is a suggestion. A model can be talked out of it by the user's own message, which is called prompt injection. A permission check in code cannot be talked out of anything.

**Validate arguments like any untrusted input.** The model produces plausible values, including ids that do not exist and amounts outside any sane range. Validate against a schema, then against business rules, before doing anything.

**Write tools need an idempotency key.** Retries happen, at the network layer and in the agent loop. A refund tool called twice with the same key must refund once. Derive the key from the request and the action, not from a random value.

**Return errors as data, not exceptions.** When a tool fails, send the model a short, factual result: "no order found with that id". It can then ask the user or try something else. A raw stack trace wastes tokens and teaches it nothing.

## Designing the tool set

The names and descriptions are part of the prompt, so they are part of the design.

- Few tools, clearly distinct. A model choosing between 40 similar tools chooses badly.
- Names that say what happens. `issue_refund` beats `process`.
- Descriptions that state when to use the tool and when not to.
- Narrow arguments. An enum with four options beats a free-text field.
- Group rarely used tools behind one tool that takes an action name, if the list grows too long.

## Trade-offs

| Pro | Con |
|-----|-----|
| The model can act on real, current data | Every tool is new attack surface |
| Structured calls remove most parsing failures | Tool definitions consume prompt tokens on every request |
| Each tool can be tested like ordinary code | The model may call the wrong tool, or none |
| Permissions stay in code, where they are enforceable | Write tools need idempotency and audit logging |

## Numbers to know

- Tool definitions cost tokens on every request. A large tool set is a recurring bill, and a reason to keep it small.
- Practical tool counts stay in the range of a handful to a dozen per agent before selection accuracy suffers.
- Each tool call adds a full model round trip, so latency grows with the number of steps.

## How to talk about it in an interview

**The sentence to say.** "The model proposes a tool call, and my code validates the arguments and checks permissions using the session identity, not anything the model produced. Write tools take an idempotency key, so a retry cannot double-charge."

Expect these follow-ups:

- What stops a user talking the model into a refund. The tool's own limits and checks, below the model.
- What you send back when a tool fails.
- How you would test tool selection. A labeled set of messages with the correct tool, scored like a classifier.

## Go deeper

- Related: [the agent loop and its bounds](agent-loop-and-bounds.md), [design a customer support agent](../questions/design-a-customer-support-agent.md)
- Course: [Grokking the AI System Design Interview](https://www.designgurus.io/course/grokking-the-ai-system-design-interview)
