# Grounding and citations

> Making every claim in an answer traceable to a retrieved source, and letting the system say it does not know.

## What it is

Grounding means the answer comes from the passages you retrieved, not from what the model remembers. A citation is the link from a specific claim back to the passage that supports it.

Together they turn an answer a user has to trust into an answer a user can check.

## The problem it solves

A language model produces fluent text whether or not it has the facts. When it fills a gap, the result reads exactly like the parts that are correct. Users cannot detect that on their own. It is why an unsourced answer over company documents is a liability rather than a feature.

Citations also change how the system fails. A wrong answer with a citation is a bug someone can report and you can trace. A wrong answer with no citation is a rumor.

## How it works

Three parts, and all three are needed.

**1. The prompt contract.** Instructions that state three rules. Answer only from the passages. Attach the source of each claim. Say plainly when the passages do not contain the answer. Keep the rules short and specific. A long list of instructions gets partly ignored.

**2. Citation plumbing.** Each retrieved passage carries an id, a source URI, and a title. The model refers to passages by id, and the application maps those ids back to real links before showing the answer.

```
[retrieved passages with ids] --> [model cites id 3] --> [UI shows the source URL for id 3]
```

Never let the model write the URL itself. It will produce URLs that look correct and do not exist. The link must come from your metadata.

**3. A verification step, when the stakes justify it.** After generating, check each claim against the passages with a second, cheaper pass. Claims with no support are removed or flagged. This costs an extra call and it catches the confident additions.

## Abstention is a feature

A system that always answers is broken. When retrieval returns nothing relevant, the correct output is "I could not find this in the documents", with the closest sources listed.

This has to be built deliberately:

- The prompt must allow it, in plain words.
- The evaluation set must contain unanswerable questions, or nothing measures it.
- The product has to accept it. If the team treats every abstention as a failure, the system learns to guess.

## Measuring it

Groundedness is a per-claim measure, not a per-answer feeling. Take a sample of answers, split them into claims, and check each claim against the cited passage. An [LLM judge](../evals/llm-as-judge.md) can do this at volume once you have measured its agreement with people.

Track two numbers separately: the share of claims that are supported, and the share of questions where the system should have abstained and did not.

## Trade-offs

| Pro | Con |
|-----|-----|
| Users can verify the answer | Citations take space in the answer and in the prompt |
| Wrong answers become traceable bugs | A verification pass adds latency and cost |
| Abstention prevents confident nonsense | Too much abstention makes the product feel useless |
| Grounding is measurable per claim | Measurement needs a labeled set, including unanswerable questions |

## Numbers to know

- A verification pass adds roughly one extra model call per answer.
- Citation metadata is small: an id, a URI, and a title per passage.
- Target the groundedness rate explicitly, for example 95 percent of claims supported, and treat it as a guardrail.

## How to talk about it in an interview

**The sentence to say.** "The prompt requires the model to answer only from the retrieved passages and to cite each claim by passage id. The application resolves those ids to real URLs, so the model never writes a link. If nothing relevant is retrieved, it says so."

Expect these follow-ups:

- What stops the model inventing a citation. Ids resolved by the application, not URLs written by the model.
- How you measure groundedness.
- What you do when two sources disagree. Prefer the newer one, show both, and report the conflict as a content problem.

## Go deeper

- The question that puts this together: [design enterprise document Q and A](../questions/design-enterprise-document-qa.md)
- Course: [Grokking the AI System Design Interview](https://www.designgurus.io/course/grokking-the-ai-system-design-interview)
