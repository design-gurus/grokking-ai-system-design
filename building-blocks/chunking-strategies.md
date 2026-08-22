# Chunking strategies

> Splitting documents into passages, which sets a ceiling on retrieval quality that no model can raise.

## What it is

A chunk is a passage of a document, small enough to embed as one vector and to fit in a prompt alongside others. Chunking is the step that decides where those passages begin and end.

## The problem it solves

A whole document is the wrong unit twice over. Its embedding averages away the one paragraph that answers the question, and it is too long to paste into a prompt with several others.

Chunks that are too small are just as bad. A passage that says "this does not apply to contractors" is useless without the rule it modifies.

## How it works

The naive method splits every N characters. It is one line of code and it cuts sentences, tables, and code blocks in half.

The method that works follows the document's own structure:

1. Split on headings and sections first, because the author already grouped related content.
2. Split long sections on paragraph boundaries until each piece fits the size target.
3. Keep a small overlap between neighbors, so a sentence at a boundary appears in both.
4. Attach metadata to every chunk: document id, version, section title, source URI, access list, updated time.

Prepending the document title and section heading to the chunk text before embedding is a cheap and reliable improvement. It gives an isolated paragraph the context the author assumed.

## Small to search, large to read

The best-performing pattern separates the two jobs.

```
embed and index:  small chunk (precise match)
send to the model: the parent section (enough context to answer)
```

Small chunks retrieve accurately because they contain one idea. Large context answers accurately because it contains the surrounding qualifications. Store the parent id on each chunk and expand at read time.

## Special cases that break naive chunking

- **Tables.** Splitting a table separates the numbers from the column headers. Keep a table whole, or repeat the header row in each piece.
- **Code.** Split on function boundaries, not on line counts.
- **Chat and ticket threads.** The question and the resolution can be far apart. Chunk the thread as a unit, or summarize it into one chunk.
- **Scanned documents.** Parsing quality caps everything downstream. Bad text extraction cannot be fixed by better chunking.

## Trade-offs

| Pro | Con |
|-----|-----|
| Small chunks give precise retrieval | They lose the context around the passage |
| Large chunks carry context | Their embeddings blur, so retrieval gets worse |
| Overlap prevents boundary losses | It inflates the index and returns near-duplicates |
| Structure-aware splitting matches how people write | It needs a parser per document type |

## Numbers to know

- Common chunk sizes: 200 to 500 tokens, with 10 to 20 percent overlap.
- One million documents at 10 chunks each is 10 million vectors. See [vector indexes](vector-indexes.md) for what that costs.
- Re-chunking a corpus means re-embedding it, so treat the chunking config as a versioned decision.
- Retrieving 5 to 10 chunks per question is typical. Measure whether the later ones help before paying for them.

## How to talk about it in an interview

The signal is that you treat chunking as a quality decision, not a preprocessing detail.

**The sentence to say.** "I would split on document structure with a small overlap. Small chunks are indexed for precision, and the parent section is sent to the model so the answer has enough context. Chunk size is a versioned config, because changing it means re-embedding the corpus."

Expect these follow-ups:

- How you would measure whether a chunking change helped. The answer is recall at k on a labeled question set, not a subjective read of a few answers.
- What happens to a document when it is edited. Re-chunk that document only, write the new chunks, delete the old ones.
- How permissions travel with a chunk. See [design enterprise document Q and A](../questions/design-enterprise-document-qa.md).

## Go deeper

The full lesson on chunking and indexing: [Grokking the AI System Design Interview](https://www.designgurus.io/course/grokking-the-ai-system-design-interview).
