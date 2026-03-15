# Semantic Spotter — Semantic Search & RAG over Policy PDFs

Semantic Spotter is an evidence-backed semantic search and Retrieval-Augmented Generation (RAG) system built over insurance policy PDFs using **LlamaIndex**. It is designed to answer policy-related questions with citations, generate high-level summaries when an LLM is available, and reduce hallucinations through guarded retrieval and similarity filtering.

## Project Overview

Insurance policy documents are dense, highly structured, and often difficult to navigate manually. This project ingests multiple insurance policy PDFs, analyzes their structure, cleans repetitive layout noise, creates smart document chunks, builds persistent indexes, and retrieves the most relevant evidence for a user query.

The system focuses on:
- semantic policy search
- evidence-backed answers with citations
- better definition lookup through smaller definition chunks
- summary generation when an LLM is enabled
- retrieval guardrails when supporting evidence is weak

## Key Features

- **Semantic retrieval over policy PDFs**
- **Cited answers** using node metadata
- **Guarded retrieval** with similarity cutoff to avoid unsupported answers
- **Definition-aware chunking** for better precision on policy terms
- **Summary support** using a SummaryIndex when an LLM is available
- **Inspector/debug mode** to review retrieved chunks and similarity scores
- **Persistent indexing** for faster subsequent runs

## Data Sources

The project works on insurance policy PDFs covering clauses, definitions, schedules, and exclusions.

### Policies used
- HDFC-Life-Easy-Health-101N110V03-Policy-Bond-Single-Pay.pdf
- HDFC-Life-Group-Poorna-Suraksha-101N137V02-Policy-Document.pdf
- HDFC-Life-Group-Term-Life-Policy.pdf
- HDFC-Life-Sampoorna-Jeevan-101N158V04-Policy-Document (1).pdf
- HDFC-Life-Sanchay-Plus-Life-Long-Income-Option-101N134V19-Policy-Document.pdf
- HDFC-Life-Smart-Pension-Plan-Policy-Document-Online.pdf
- HDFC-Surgicare-Plan-101N043V01.pdf
- Principal-Sample-Life-Insurance-Policy.pdf

## Architecture

The pipeline follows this flow:

**PDF ingestion → structure analysis → cleaning → smart chunking → indexing → guarded retrieval → routing → answer with citations + inspector**

### Layer 1 — PDF Ingestion and Extraction
Extracts text line by line with layout signals so section structure and formatting are preserved as much as possible.

### Layer 2 — Structure Analysis and Cleaning
Detects headings, tables, and table-of-contents patterns, while removing repetitive headers and footers that would otherwise pollute embeddings.

### Layer 3 — Smart Chunking
Uses section-aware or paragraph-aware chunking depending on the document. Definitions are chunked more aggressively to improve lookup precision for policy terms.

### Layer 4 — Indexing
Builds a persistent **VectorStoreIndex** for semantic retrieval and a **SummaryIndex** for broad document-level summaries.

### Layer 5 — Guarded Retrieval and Routing
Retrieves top-k chunks and applies a similarity cutoff. If evidence quality is too weak, the system returns **"Not found in documents"** instead of hallucinating an answer.

### Layer 6 — Answer, Citations, and Inspector
Returns the final answer with citations. The inspector shows retrieved chunks, scores, and filtered results for debugging and quality checks.

## Core Configuration

```python
OUT_DIR = 'outputs/pdf_analysis'
PERSIST_DIR = 'storage_llamaindex'
REPORT_DIR = 'outputs/report_artifacts'

DEFAULT_TARGET_CHARS = 2400
DEFAULT_OVERLAP_CHARS = 300
DEFN_TARGET_CHARS = 1400
DEFN_OVERLAP_CHARS = 220

TOP_K = 6
SIMILARITY_CUTOFF = 0.8
```

## Recommended Project Structure

```text
.
├── data/
│   └── pdfs/
├── outputs/
│   ├── pdf_analysis/
│   └── report_artifacts/
├── storage_llamaindex/
├── requirements.txt
└── main.py / notebook
```

## Installation

1. Create and activate a Python virtual environment.
2. Install dependencies:

```bash
pip install -r requirements.txt
```

## How to Run

1. Place all policy PDFs inside `data/pdfs/`.
2. Run the main script or notebook to:
   - ingest PDFs
   - analyze structure
   - clean extracted text
   - chunk and index documents
   - save evidence JSONs and report artifacts
3. Query the system using the helper methods.

### Example workflow

```python
answer = ask("What are the exclusions under this policy?")
print(answer)

inspect_query("What is the waiting period?")
```

## Outputs

### Generated artifacts
- **Evidence JSONs** in `outputs/pdf_analysis`
- **Persistent index** in `storage_llamaindex`
- **Flowcharts / report artifacts** in `outputs/report_artifacts`

## Challenges and Mitigations

### 1. Repetitive headers and footers
Repeated boilerplate text can dominate embeddings and distort retrieval quality.

**Mitigation:** repetition-based detection and removal during preprocessing.

### 2. Tables and schedules
Insurance documents often store important facts in tabular form.

**Mitigation:** preserve tables as table chunks for better factual lookup.

### 3. Definition precision
Policy definitions are short and sensitive to wording.

**Mitigation:** use smaller chunk sizes for definition-heavy sections.

### 4. Hallucination risk
Generic LLM-style answers can become unsafe in regulated document settings.

**Mitigation:** guarded retrieval with similarity cutoff and fallback response when evidence is insufficient.

### 5. PDF variability
Different documents vary in heading patterns, spacing, and layout.

**Mitigation:** adaptive chunking strategy based on document structure.

### 6. Debuggability
RAG systems are hard to trust without visibility into retrieved evidence.

**Mitigation:** add an inspector utility to review raw hits and similarity scores.

## Limitations

- Performance depends on extraction quality from PDFs.
- Summary generation depends on LLM availability.
- Retrieval quality may vary for highly scanned or poorly formatted documents.
- Similarity cutoff may need tuning for new document collections.

## Future Improvements

- Add OCR support for scanned PDFs
- Support more insurance providers and policy families
- Add hybrid retrieval (keyword + vector)
- Add policy comparison view
- Add UI for interactive querying and evidence exploration
- Add evaluation benchmarks for citation quality and retrieval accuracy

## Tech Stack

- Python
- LlamaIndex
- Vector indexing and semantic retrieval
- PDF parsing and structure-aware preprocessing
- Optional LLM-based summarization

## Author

**Nikesh Kumar**

---

This project demonstrates how a domain-focused RAG pipeline can be built for high-trust document search, especially in policy-heavy domains where evidence, traceability, and hallucination control are critical.
