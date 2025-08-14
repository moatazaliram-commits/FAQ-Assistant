# Smarter FAQ & Product Assistant

> A product-aware semantic FAQ and response system that returns sourced answers from a knowledge base and only generates contextual replies when no good source exists.

---

## Project overview

This repository contains an implementation (originally in `FAQ.ipynb`) of a **dynamic FAQ / semantic search** system that:

- Ingests FAQ and product data (CSV / DataFrame).
- Cleans and preprocesses text (`preprocess_text`).
- Generates vector embeddings using a sentence encoder (e.g., `SentenceTransformer`).
- Indexes embeddings using FAISS for fast nearest-neighbor retrieval.
- Exposes a `SemanticSearchEngine` to perform product-scoped and global searches (e.g., `search_by_product(asin, query)` with fallback `search_similar_questions(query)`).
- Wraps search and LLM generation into a `DynamicFAQSystem` that composes final answers (the notebook referenced a `gemini_model` as the LLM).
- Stores metadata and Q&A items in a simple `FAQDatabase`.

The original development notebook path: `/mnt/data/FAQ.ipynb`.

---

## Architecture (high-level)

- **Data sources**: QnA CSVs, product metadata.
- **Preprocessing**: `preprocess_text` (clean → tokenize → lemmatize/normalize).
- **Embeddings**: SentenceTransformer or equivalent.
- **Index**: FAISS index built from normalized embeddings.
- **Search layer**: `SemanticSearchEngine` (product-aware search + fallback).
- **Orchestrator**: `DynamicFAQSystem` which uses the search engine and the LLM to generate answers when needed.
- **Outputs**: user-facing API, notebook demo, and downloadable assets (e.g., diagrams).

Mermaid flowcharts are included in the repo (or provided in the project notes). In Overleaf upload the generated PNGs into `images/` (see “Mermaid & Overleaf” below).

---

## Quick demo / usage (copy-ready examples)

> These examples assume you have refactored the notebook into script/module form (e.g., `faq_system.py`) or are running the notebook interactively.

### 1) Initialize the system (example)
```python
# load your data into a pandas DataFrame `df` with columns such as:
# "question", "answer", "asin" (optional), "source", etc.

from faq_system import initialize_system, DynamicFAQSystem

# Example: builds preprocessing, embeddings, FAISS index and returns system objects
search_engine, faq_db, dynamic_system = initialize_system(df)
# or if initialize_system returns a single orchestrator:
# dynamic_system = initialize_system(df)
```
### 2) Simple product-scoped query
```python
# If you have product context (ASIN), prefer product-scoped search:
asin = "B0XXXXX"            # example product identifier / SKU
query = "How do I reset the device to factory settings?"

results = search_engine.search_by_product(asin, query)
if not results:
    # fallback to global/similar questions
    results = search_engine.search_similar_questions(query)

# Results typically contain: [(question, answer, score, source), ...]
```
### 3) High-level ask via orchestrator
```python
# If you use DynamicFAQSystem to handle end-to-end logic:
response = dynamic_system.ask(query, asin=asin, k=5)
# response could include: text, source_links, confidence_score, used_fallback (bool)
```
## Installation & dependencies
Install
```bash
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```
Note: On some systems faiss installation differs (e.g., pip install faiss-cpu on Linux). If you use GPU FAISS, install faiss-gpu and the corresponding CUDA toolchain.
