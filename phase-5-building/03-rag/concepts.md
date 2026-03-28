# Concepts: RAG

> **Tag: core** — Retrieval-Augmented Generation is how you give a model access to information it was never trained on, without retraining it.

---

## Chunking Strategy

**What it is** — The process of splitting source documents into smaller text segments before embedding and storing them for retrieval.

**Why it matters** — Embedding models have input limits (typically 512–8,192 tokens), and retrieval works best when chunks are semantically focused. Chunks that are too large match too broadly; chunks that are too small lose context and produce unhelpful snippets.

**How it works** — Chunking happens at index time — before the user ever asks a question. You split each document into segments and store each segment separately in the vector store.

Three common approaches:

**Fixed-size chunking** — split by token or character count:
```
chunks = split_by_tokens(document, size=300, overlap=50)
# overlap: each chunk shares 50 tokens with the previous one
# WHY: overlap preserves context at boundaries — a sentence that
# straddles two chunks is fully present in at least one of them
```

**Structural chunking** — split by document structure (paragraphs, headings, sections):
```
chunks = split_by_structure(document, unit="paragraph")
# preserves semantic meaning at natural boundaries
```

**Semantic chunking** — split when topic shifts (using a secondary model to detect breaks):
```
chunks = split_by_semantic_similarity(document, threshold=0.7)
# computationally expensive; best for highly heterogeneous documents
```

The right choice depends on document type. For structured documents (docs, FAQs, manuals): structural chunking. For unstructured prose: fixed-size with overlap. For highly varied content: semantic.

**Production tip** — Always store the chunk's source metadata alongside it: document ID, section heading, page number, and URL. Without metadata, retrieved chunks cannot be cited and their provenance cannot be verified.

**Common failure** — Chunks that are too large return broad, weakly relevant matches. Chunks that are too small (individual sentences) lose context and return fragments that don't make sense in isolation. Tune chunk size empirically on a sample of your real documents — the right size varies by domain.

---

## Embedding and Vector Search

**What it is** — The process of converting text into a dense numerical vector (embedding) that captures its meaning, and searching for vectors that are close to a query vector.

**Why it matters** — Traditional keyword search finds exact word matches. Embedding search finds semantically similar content — "how do I cancel my subscription" matches "cancellation policy" even though they share no keywords.

**How it works** — The pipeline has two phases:

**Index time** (done once, upfront):
```
for chunk in all_chunks:
  vector = embedding_model.encode(chunk.text)  # dense float array, ~1024 dimensions
  vector_store.insert(chunk.id, vector, metadata=chunk.metadata)
```

**Query time** (on every user question):
```
query_vector = embedding_model.encode(user_query)
results = vector_store.search(query_vector, top_k=5)
# returns the 5 chunks whose vectors are closest to the query vector
# "closeness" = cosine similarity or dot product
```

The embedding model must be the same at both index time and query time. Using different models for indexing vs. querying produces garbage results — the vector spaces are incompatible.

Similarity is measured in the vector space — not by content length, not by keyword frequency. A chunk from a FAQ about "account deletion" will score high on a query about "how to close my account" because their embeddings are nearby in the semantic space.

**Production tip** — Use the same embedding model version for indexing and querying, and lock it in configuration. An embedding model upgrade requires re-indexing all documents — applying it only to new queries silently breaks retrieval for the entire existing index.

**Common failure** — Mixing embedding models between index time and query time (usually by upgrading the model mid-operation without re-indexing). Retrieval silently degrades — similarity scores become meaningless because the query vector and the stored vectors are in different spaces.

---

## Retrieval Quality

**What it is** — The accuracy and relevance of the chunks returned by the vector search, measured by how well they match what the user actually asked.

**Why it matters** — A RAG system is only as good as its retrieval. If the retrieved chunks are irrelevant, the model will either hallucinate an answer (because it has no good context) or correctly say "I don't know" (unhelpful but honest). Neither is the goal.

**How it works** — Retrieval quality is improved through three techniques:

**Query rewriting** — reformulate the user's question before embedding it:
```
# User asks: "it keeps crashing when I open large files"
# Rewritten: "application crash when opening large files, memory error"
# The rewritten query produces better embedding matches
rewritten_query = model.chat(rewrite_prompt(user_query))
query_vector = embedding_model.encode(rewritten_query)
```

**Re-ranking** — use a second model to score retrieved chunks after the initial vector search:
```
candidates = vector_store.search(query_vector, top_k=20)  # broad initial retrieval
ranked = reranker.score(user_query, candidates)
top_chunks = ranked[:5]  # keep only the top 5 after re-ranking
```

Re-rankers are smaller, faster models that compare a query-chunk pair directly — more accurate than vector similarity alone.

**Hybrid search** — combine vector search with keyword search:
```
vector_results  = vector_store.search(query_vector, top_k=10)
keyword_results = keyword_index.search(user_query, top_k=10)
merged = merge_and_deduplicate(vector_results, keyword_results)
```

For most production RAG systems, hybrid search + re-ranking delivers significantly better retrieval quality than vector search alone.

**Production tip** — Evaluate retrieval quality separately from generation quality. Build a small test set of question-answer pairs with known source chunks, and measure what percentage of the time the correct chunk appears in the top-5 results. Improving retrieval is usually higher leverage than improving prompts.

**Common failure** — Returning the model's final answer when retrieval is failing. The model will often generate a confident-sounding answer even when none of the retrieved chunks are relevant, because the model's training data contains general knowledge. The output looks fine but is not grounded in your documents.

---

## Context Injection

**What it is** — The step that takes retrieved chunks and inserts them into the messages array before the model call, so the model can use them to answer the question.

**Why it matters** — Retrieved chunks are useless unless they're in the prompt. How you inject them — where in the messages array, how they're labeled, how many you include — directly affects the quality of the model's answer.

**How it works** — Retrieved chunks are injected as part of the user message or as a dedicated context block:

```
# Build a context block from retrieved chunks
context_block = ""
for i, chunk in enumerate(top_chunks):
  context_block += "[Source " + (i+1) + ": " + chunk.source + "]\n"
  context_block += chunk.text + "\n\n"

# Inject into the user message
augmented_message = """
Use the following context to answer the question.
If the answer is not in the context, say so.

Context:
""" + context_block + """

Question: """ + user_query

messages = [
  { role: "system", content: system_prompt },
  { role: "user",   content: augmented_message }
]
```

The token budget must account for the injected context. (See: Module 01 — Token budgeting.)

Key decisions:
- How many chunks to inject (typically 3–8; more is not always better)
- Whether to include source metadata alongside the chunk text (improves citation accuracy)
- Where to place the context (before the question, not after — models attend better to content at the end)

**Production tip** — Label each injected chunk with a source identifier (e.g., `[Source 1: docs/billing.md]`). This enables citation extraction from the model's response and gives you a paper trail for debugging incorrect answers.

**Common failure** — Injecting so many chunks that the combined context plus the user's question exceeds the context limit. The call fails with a 400 error. Fix: calculate the token budget available for context before retrieval, set `top_k` to a number that fits within that budget, and truncate chunks if necessary.

---

## Answer Grounding and Citation

**What it is** — The practice of ensuring the model's answer is based on the retrieved context, and that the user can verify which source produced which claim.

**Why it matters** — Without grounding, a model with strong general knowledge will answer from training data rather than your documents, producing confident answers that are accurate in general but wrong for your specific use case. Without citation, users cannot verify answers and you cannot audit failures.

**How it works** — Grounding is enforced in the system prompt:

```
system_prompt = """
Answer questions using ONLY the provided context.
If the context does not contain the answer, respond with:
"I couldn't find information about that in the documentation."
Do not use your general knowledge to fill gaps.
"""
```

Citation is extracted from the model's response:

```
# Ask the model to include source references in its answer
user_message = """...[context with [Source N] labels]...

Question: How do I export data?
Include [Source N] references for each claim.
"""

# Parse citations from the response
response_text = "You can export via Settings > Export [Source 2]."
citations = extract_citations(response_text)  # returns ["Source 2"]
cited_chunks = [top_chunks[i] for i in citations]
```

Some providers support structured output that includes citations as a separate field — use it when available.

**Production tip** — Always ask the model to say "I don't know" rather than guess when context is insufficient. A confident wrong answer based on training data is harder to debug than an honest "I couldn't find that." Use the grounding instruction explicitly; it is not applied by default.

**Common failure** — The model answers from training data when retrieved context is insufficient. This looks correct for general questions but produces wrong answers for domain-specific questions (your product's specific behavior, your company's specific policies). The fix is the explicit "only use the provided context" instruction.

---

## Retrieval Failure Handling

**What it is** — The logic that decides what to do when the vector search returns no results, or returns results that are too dissimilar to the query to be useful.

**Why it matters** — Retrieval will fail on questions outside your document corpus, poorly worded queries, or gaps in your knowledge base. Without explicit handling, the model generates an answer from training data — confidently wrong, and hard to detect.

**How it works** — Handle failure at two points:

**Similarity threshold check** — reject retrievals that are too dissimilar:
```
MIN_SIMILARITY = 0.7  # tune based on your embedding model and domain

results = vector_store.search(query_vector, top_k=5)
relevant_chunks = [r for r in results if r.similarity >= MIN_SIMILARITY]

if len(relevant_chunks) == 0:
  return fallback_response(user_query)
```

**Fallback response** — define what the model should return when it has no useful context:
```
def fallback_response(query):
  return """
  I couldn't find relevant information in the documentation to answer your question.
  You may want to contact support or check [support link].
  """
```

The threshold value requires calibration. Too high: many valid questions get fallback responses. Too low: irrelevant chunks get injected, producing hallucinated answers. Start at 0.6–0.7 and adjust based on observed failure modes.

**Production tip** — Log every retrieval failure with the user's query and the similarity scores of the top results. This is your knowledge base gap detector — recurring failed queries reveal what documentation is missing.

**Common failure** — Not distinguishing between "no results" (empty retrieval) and "low-quality results" (results returned but below threshold). Both cases need the fallback path, but "no results" means the query is outside your corpus entirely, while "low-quality results" might mean the query needs rewriting. Log both separately to understand which you're hitting.

---

## Further reading

**RAG from scratch** — [https://github.com/langchain-ai/rag-from-scratch](https://github.com/langchain-ai/rag-from-scratch)
A code-first walkthrough of each RAG component. Language-specific (Python) but the components map directly to the cards above.

**Embedding model benchmarks** — [https://huggingface.co/spaces/mteb/leaderboard](https://huggingface.co/spaces/mteb/leaderboard)
MTEB leaderboard — the standard benchmark for retrieval embedding models. Use this for model selection.

**Chunking strategies in depth** — [https://www.pinecone.io/learn/chunking-strategies/](https://www.pinecone.io/learn/chunking-strategies/)
Practical comparison of chunking approaches with retrieval quality tradeoffs.

**Re-ranking overview** — [https://www.mixedbread.ai/blog/reranking](https://www.mixedbread.ai/blog/reranking)
What re-ranking models do and when they improve retrieval quality.
