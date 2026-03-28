# Basic Example: RAG

> Pseudocode walkthrough — no real SDK imports. The goal is to understand
> the two-phase RAG pipeline: index time and query time.

---

## Scenario

A knowledge base Q&A system for a software product. Users can ask questions about the product, and the system answers using the company's internal documentation — not the model's training data.

---

## Phase 1: Index time (run once, when documents are loaded)

This phase runs when documents are added to the knowledge base, not on every user query.

### Step 1: Load and chunk the documents

```
all_chunks = []

for document in knowledge_base.documents:
  # WHY: split documents into smaller pieces before embedding.
  # The embedding model has input limits, and smaller chunks produce
  # more precise retrieval matches.
  chunks = split_by_tokens(
    document.text,
    size=300,    # target size per chunk in tokens
    overlap=50   # each chunk shares 50 tokens with the previous one
                 # WHY: overlap preserves context at chunk boundaries
  )

  for i, chunk_text in enumerate(chunks):
    all_chunks.append({
      id:       document.id + "_chunk_" + i,
      text:     chunk_text,
      source:   document.filename,   # WHY: store source for citation later
      section:  document.section
    })
```

### Step 2: Embed each chunk and store it

```
for chunk in all_chunks:
  # WHY: embed the text into a dense vector that captures its meaning.
  # Semantically similar text will produce nearby vectors.
  vector = embedding_model.encode(chunk.text)

  # WHY: store both the vector (for search) and the metadata (for context injection).
  vector_store.insert(
    id=chunk.id,
    vector=vector,
    metadata={ text: chunk.text, source: chunk.source, section: chunk.section }
  )

print("Indexed " + len(all_chunks) + " chunks")
```

---

## Phase 2: Query time (runs on every user question)

### Step 3: Embed the user's question

```
user_query = get_user_input()

# WHY: embed the question using the SAME model used at index time.
# Using a different model would make the query vector incompatible
# with the stored chunk vectors.
query_vector = embedding_model.encode(user_query)
```

### Step 4: Retrieve the most relevant chunks

```
# WHY: search for the 5 chunks whose vectors are most similar to the query.
# "Similar vectors" means "similar meaning" — not matching keywords.
results = vector_store.search(query_vector, top_k=5)

# results is a list of chunks, ordered by similarity (highest first)
top_chunks = results
```

### Step 5: Inject context and call the model

```
# WHY: label each chunk with a source reference.
# This enables the model to include citations in its answer.
context_block = ""
for i, chunk in enumerate(top_chunks):
  context_block += "[Source " + (i+1) + ": " + chunk.source + "]\n"
  context_block += chunk.text + "\n\n"

augmented_message = """
Use the following context to answer the question.
If the answer is not in the context, say so — do not use general knowledge.

Context:
""" + context_block + """

Question: """ + user_query

response = model.chat({
  model: "claude-sonnet-4-6",
  messages: [
    { role: "system", content: "You are a helpful product documentation assistant." },
    { role: "user",   content: augmented_message }
  ],
  max_tokens: 500
})

print(response.content)
```

---

## What this example shows

This walkthrough demonstrates four of the six component cards from `concepts.md`:

- **Chunking strategy** — fixed-size chunking with overlap, storing source metadata
- **Embedding and vector search** — same model at index time and query time; vector similarity search
- **Context injection** — labeled context block in the user message, with source references
- **Answer grounding** — explicit instruction to use only the provided context

It deliberately leaves out:

- **Retrieval quality** — no re-ranking or query rewriting. The basic example uses raw vector similarity, which is sufficient for demonstration but often not for production.
- **Retrieval failure handling** — if no relevant chunks are found, or all similarity scores are low, the model will still receive an empty or irrelevant context block. See `production.md` for the fallback path.
