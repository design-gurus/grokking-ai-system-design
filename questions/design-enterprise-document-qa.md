# Design enterprise document Q and A

> Answer employee questions from a company's own documents, with citations, and never show a document the person is not allowed to read.

**Family:** GenAI and LLM product
**Also asked as:** design a RAG system, design internal search with AI, design a knowledge assistant.

## 1. Frame the problem

An employee asks a question in plain language. The system finds the relevant passages in the company's documents and generates an answer that cites them.

The task is retrieval plus grounded generation. Grounded means every claim in the answer traces to a retrieved passage. The model is not the source of truth. The corpus is.

Scale to assume: 50,000 employees, 5 million documents from wikis, tickets, and file shares, 100,000 questions a day. Documents change all day. Permissions vary per document.

This is the most common real deployment of an LLM inside a company. One requirement makes it hard: permissions.

## 2. Define success

| Metric type | Metric | Target |
|-------------|--------|--------|
| Offline, retrieval | Recall at 10 on a labeled question set | 0.9 or higher |
| Offline, answer | Groundedness and correctness, graded by a judge and spot-checked by people | 95 percent grounded |
| Online | Deflection rate, thumbs up rate, repeat use | Stated per product |
| Guardrail | Zero permission leaks, p95 time to first token under 1.5 seconds | Absolute |

Split retrieval quality from answer quality, and say why. If the answer is wrong, you need to know whether the passage was missing or the model ignored it. One combined score cannot tell you, so it cannot guide the next fix.

Permission leaks are not a metric with a target. They are a hard failure, tested like a security control.

## 3. Map the data

This system is two pipelines with different shapes. Say that early.

| Pipeline | Shape | Latency |
|----------|-------|---------|
| Ingestion | Batch and stream, high throughput | Minutes |
| Query | Request path, low latency | Seconds |

**Ingestion.** Connectors pull from each source or subscribe to its change events. Documents are parsed, split into chunks, embedded, and written to a vector index plus a document store.

Chunking decides most of the answer quality. Split on structure first, headings and sections, not on a fixed character count. Keep a small overlap so a sentence at a boundary is not lost. Store the parent document id with each chunk so the answer can widen the context when a chunk is too small on its own.

**Permissions travel with the chunk.** Copy the source access control list onto every chunk at ingestion, and re-sync it when it changes on the source. Permission changes are their own event stream, separate from content changes, and they are usually more urgent.

**Freshness.** On update, re-chunk only that document, write the new chunks, and delete the old ones. Make writes idempotent, keyed on document id and version, so a retry does not duplicate chunks.

## 4. Pick the model

Three model choices, and only one of them is interesting.

- **Embedding model.** Start with a strong general model. Measure recall on your own labeled questions before considering a domain-specific one. Changing this later means re-embedding all 5 million documents, so treat it as a versioned decision.
- **Reranker.** A cross-encoder that scores the query against each candidate passage together, rather than comparing two separate vectors. It is far more accurate and far slower, so it runs over 50 candidates, not 5 million.
- **Generator.** A hosted model to start. The interesting part is not which one. It is the prompt contract: answer only from the passages, cite each claim, and abstain when the passages do not contain the answer.

Fine-tuning is the wrong first answer here. The gap is knowledge, and the knowledge changes daily. See [RAG vs fine-tuning vs prompting](../cheat-sheets/rag-vs-fine-tuning-vs-prompting.md).

## 5. The serving path

Budget: 1.5 seconds to first token, a few seconds to a complete answer.

```
[question] --> [embed: 20 ms]
           --> [hybrid retrieval with permission filter: 50 ms]
           --> [rerank top 50 to top 8: 150 ms]
           --> [assemble prompt within context budget]
           --> [generate, streaming, with citations]
```

**Hybrid retrieval, always.** Vector search finds meaning and misses exact strings. Employees search for error codes, ticket numbers, and product names, which is exactly what keyword search is good at. Run both and merge the ranked lists.

**Filter inside the query, never after.** Filtering after retrieval means a user with narrow access gets an empty answer while matching documents exist. Response timing can also reveal that a document exists. Push the access list into the index query, or partition indexes by tenant or by group. This is the single most common follow-up in this question.

**Prompt assembly is a budget problem.** You have a context limit, and passages compete with conversation history. Take the top passages until the budget is spent, keeping at most two per document so one long document cannot fill the whole prompt.

## 6. Keep it working

- **Log every question, the retrieved chunk ids, and the answer.** Without the chunk ids you cannot debug a bad answer later.
- **Thumbs down is a retrieval bug most of the time.** Route a sample to review and label whether the passage was missing or the model ignored it.
- **Watch for stale answers.** A deleted document that is still in the index will be cited confidently. Deletion propagation is a correctness feature, and it should be alerted on, not just scheduled.
- **Re-embedding is a project.** Build the new index alongside the old one, compare recall on the labeled set, then cut over.
- **Evaluation runs on a schedule**, not once. A fixed set of questions with known answers, run nightly, catches regressions from a prompt edit or a model version change. See [evals/](../evals/).

## 7. Count the costs

Per question: about 4,000 input tokens of passages and instructions, about 300 output tokens. At 100,000 questions a day, that is roughly 400 million input tokens a day. Input tokens dominate, which is the opposite of a chat product, and it points straight at the levers.

Levers: retrieve fewer passages, after measuring how many actually help. Cache repeated answers, and cache the fixed instruction prefix. Route short factual questions to a smaller model. Ingestion cost is a one-time embedding pass plus a small daily delta, which is usually far smaller than the query bill.

## Common follow-ups

- **How do you handle a question the corpus cannot answer?** The prompt requires "I do not know", and evaluation must include unanswerable questions. A system that never abstains is broken.
- **How do you cite?** Map each returned passage to its source URI and show it next to the claim. Citation is a retrieval feature, not a model feature.
- **What if two documents disagree?** Prefer the newer one, surface both, and treat conflicts as a content problem to report, not a model problem to hide.
- **How do you support conversation?** Rewrite the follow-up question into a standalone query before retrieval. "What about for contractors?" retrieves nothing on its own.

## Common mistakes

- Filtering permissions after retrieval.
- Fixed-size chunking with no overlap and no structure.
- One quality score, so you cannot tell retrieval failures from generation failures.
- Proposing fine-tuning for a corpus that changes daily.
- No plan for deletes.

## Go deeper

The full worked solution, including the enterprise capstone: [Grokking the AI System Design Interview](https://www.designgurus.io/course/grokking-the-ai-system-design-interview).

Written for a general system design round, architecture first: [Design a RAG pipeline](https://github.com/design-gurus/grokking-system-design/blob/main/questions/design-rag-pipeline.md).
