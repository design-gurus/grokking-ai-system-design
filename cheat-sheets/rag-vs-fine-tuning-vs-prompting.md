# RAG vs fine-tuning vs prompting

> The most common follow-up in an LLM design question, decided in one table.

Almost every GenAI design question reaches the same fork. The model does not do what the product needs. You have three ways to fix that, and interviewers want to hear you pick one for a reason, not by fashion.

First, the definitions, in one line each.

- **Prompting** means changing the instructions you send with each request. Nothing about the model changes.
- **RAG** means retrieval-augmented generation. You search a corpus, then paste the best passages into the prompt.
- **Fine-tuning** means training the model further on your own examples, so the new behavior is inside the weights.

## The decision table

| Question | Prompting | RAG | Fine-tuning |
|----------|-----------|-----|-------------|
| Fixes missing knowledge? | No | Yes | Poorly |
| Fixes wrong format or tone? | Yes | No | Yes |
| Fixes a skill the model lacks? | Sometimes | No | Yes |
| Handles data that changes daily? | No | Yes | No |
| Can cite a source? | No | Yes | No |
| Time to first version | Hours | Days | Weeks |
| Cost per request | Baseline | Higher, longer prompts | Lower, shorter prompts |
| Cost to change | Zero | Reindex | Retrain |
| Needs labeled examples? | No | No | Yes, hundreds to thousands |

## The shortcut

Ask one question: **is the gap knowledge or behavior?**

Knowledge gaps mean the model does not know a fact. Your policy documents, this week's prices, a customer's order history. Retrieval fixes knowledge. Training does not, and training on facts is the classic wrong answer in this round, because the facts change the day after the run finishes.

Behavior gaps mean the model knows enough but answers wrong. Wrong format, wrong tone, ignores the schema, too long. Prompting fixes most behavior gaps. Fine-tuning fixes the ones that survive a good prompt, and it is worth the cost when the prompt needed for that behavior is very long.

## What to say in an interview

Say the ladder, in order, and stop where the problem stops.

1. Prompt an existing model. Measure it. This is the baseline every other option is compared against.
2. Add retrieval when the answer depends on data the model was never trained on.
3. Fine-tune when a good prompt still fails, or when the prompt is so long that shortening it saves real money.

**The sentence to say.** "I would start with prompting plus retrieval, because the content changes weekly and users need citations. I would only fine-tune if evaluation shows the model still gets the format wrong after prompt work. Even then I would fine-tune a small model on formatting, not on the knowledge."

## They combine

These are not exclusive. The common production shape is all three: a fine-tuned small model, given retrieved passages, under a carefully written prompt. Say that if the interviewer pushes.

A useful pairing: fine-tune for format and tone, retrieve for facts. Each tool does the job it is good at.

## Where each one fails

**Prompting fails silently at scale.** A prompt that works on ten examples fails on the eleventh, and you only learn that from evaluation. Long prompts also cost money on every single request. See [numbers you should know](ai-numbers.md).

**RAG fails at chunking and permissions.** If the retrieval step returns the wrong passage, no model fixes the answer. If permissions are filtered after retrieval instead of inside the query, users can be told that a document exists. That is a real leak, and it is the most common follow-up in enterprise questions.

**Fine-tuning fails at maintenance.** Every base model upgrade means running the job again. Your data must stay clean and versioned. Teams that fine-tune early often end up stuck on an old base model because redoing the work is expensive.

## Go deeper

The full lesson, with the evaluation setup that decides between them: [Grokking the AI System Design Interview](https://www.designgurus.io/course/grokking-the-ai-system-design-interview).
