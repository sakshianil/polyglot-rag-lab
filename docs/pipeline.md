# RAG Pipeline Documentation

This document explains the pipeline used in this project and how each folder supports parsing, chunking, embedding, vector database storage, retrieval, and answer generation.

## Objective

The goal is to demonstrate a practical Basic and Advanced RAG workflow that can work with multiple document types and answer questions in more than one language.

## Pipeline overview

```text
Input files
  ↓
Document parser / loader
  ↓
Text extraction and metadata creation
  ↓
Chunking
  ↓
Embedding generation
  ↓
Vector database storage
  ↓
Retriever
  ↓
Prompt with retrieved context
  ↓
LLM answer
  ↓
Structured answer for documentation or notebooks
```

## Input modalities

| Modality | Example in this repo | How it becomes RAG-ready |
|---|---|---|
| Markdown | `data/raw/markdown/german_vegan_food_guide.md` | Loaded directly as text |
| Text | `data/raw/text/audio_transcript_sample.txt` | Treated as transcript text |
| CSV | `data/raw/tables/european_vegan_recipes.csv` | Converted row-by-row into text records |
| Image caption | `data/raw/multimodal/image_caption_samples.csv` | Represents output from an image captioning model |
| Audio | Transcript sample | Speech-to-text output is embedded |
| Video | Caption/transcript sample | Frames and transcript become text chunks |
| PDF | Not included as binary sample | Can be parsed with LlamaIndex or PyMuPDF |

## Step 1 — Parse documents

A parser reads each file and converts it into a standard document object.

Recommended metadata fields:

```json
{
  "source": "data/raw/markdown/german_vegan_food_guide.md",
  "modality": "markdown",
  "language": "en",
  "topic": "German vegan food"
}
```

## Step 2 — Normalize text

Before chunking, text should be cleaned:

- Remove unnecessary whitespace
- Keep headings where useful
- Preserve table row meaning
- Attach source metadata
- Keep language hints when available

## Step 3 — Chunk documents

Chunking turns long documents into retrievable units.

Recommended settings for small demos:

```python
chunk_size = 512
chunk_overlap = 50
```

For large documents, tune chunk size based on answer quality and retrieval precision.

## Step 4 — Generate embeddings

Each chunk is transformed into an embedding vector. The embedding model should support semantic similarity and ideally multilingual retrieval.

For multilingual RAG, choose an embedding model that performs well across languages.

## Step 5 — Store vectors

The vector database stores:

- Embedding vectors
- Chunk text
- Source file path
- Document metadata
- Language or modality tags

The folder `data/vector_store/` is included as a placeholder. Generated vector DB files are ignored by Git because they can become large.

## Step 6 — Retrieve context

When the user asks a question, the retriever finds the most relevant chunks.

Example:

```text
Question: Was sind typische deutsche Gerichte?
Relevant chunk: Typical German dishes include Kartoffelsalat, Sauerkraut, Bratwurst, Schnitzel, Spätzle...
```

## Step 7 — Generate answer

The retrieved chunks are inserted into the prompt so the LLM answers from the knowledge base.

A good RAG prompt should ask the model to:

- Use only retrieved context when possible
- Answer in the user's language
- Be structured and readable
- Mention if the context is insufficient
- Prefer citations or source names when available

## Basic RAG vs Advanced RAG

| Feature | Basic RAG | Advanced RAG |
|---|---|---|
| Loading | Simple document loading | Multiple formats and metadata |
| Retrieval | Top-k vector search | Hybrid search, reranking, filters |
| Prompting | Direct context injection | Context compression and structured prompts |
| Language | Single-language focus | Multilingual query handling |
| Evaluation | Manual checking | Retrieval and answer quality metrics |
| UI | Simple Gradio Q&A | Advanced Gradio app with controls and source display |

## Suggested next upgrades

1. Add source citations in the Gradio output.
2. Add a retrieval preview panel showing the selected chunks.
3. Add hybrid retrieval using BM25 plus vector search.
4. Add reranking before final answer generation.
5. Add multilingual embedding benchmark examples.
6. Add evaluation notebooks for faithfulness and retrieval precision.
