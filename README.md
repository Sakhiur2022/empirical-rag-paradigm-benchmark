# Benchmarking Database Paradigms for Retrieval-Augmented Generation: A Workload-Specific Performance Analysis

**Authors:** A.M.A Sakhiur Rahman • Sadman Jawad Suborno  
**Affiliation:** North South University  
**Course:** CSE498R Directed Research

---

## Overview

This repository contains the full experimental codebase for our benchmark study comparing three database paradigms for the retrieval stage of Retrieval-Augmented Generation (RAG) pipelines:

| Paradigm                          | System                      |
| --------------------------------- | --------------------------- |
| Relational (SQL + vector bolt-on) | Supabase + pgvector         |
| Document NoSQL + vector hybrid    | MongoDB Atlas Vector Search |
| Purpose-built vector DB           | Qdrant                      |

The central novelty is **workload-specific characterization**: rather than reporting aggregate performance, we profile each system across three distinct RAG workload types and derive a per-paradigm trade-off signature.

---

## Research Question

> How do relational, document-oriented NoSQL, and purpose-built vector database paradigms differ in retrieval effectiveness and latency across distinct RAG workload types — short-context factual queries (W1), multi-hop reasoning queries (W2), and semantic similarity queries (W3) — when evaluated against a single domain-specific corpus?

---

## Workload Taxonomy

| ID  | Type                  | Definition                                              | Primary Retrieval Signal          |
| --- | --------------------- | ------------------------------------------------------- | --------------------------------- |
| W1  | Short-context factual | Single-hop, lookup-style; answer contained in one chunk | Keyword + semantic match          |
| W2  | Multi-hop reasoning   | Answer requires combining 2+ chunks across documents    | Semantic + cross-document linking |
| W3  | Semantic similarity   | Vague, conceptual queries; no single correct answer     | Pure embedding similarity         |

---

## Corpus

~50,000 ArXiv CS abstracts and full papers. Selected for high semantic density and availability of clear ground-truth relevance judgments.

---

## Metrics

**Retrieval effectiveness (primary):** NDCG@10 • MRR • Precision@10 • Recall@10  
**Operational (secondary):** Query latency (p50/p95/p99 ms) • Indexing time • Memory footprint at rest

---

## Experimental Controls

All three systems are evaluated under identical conditions:

- Same hardware for every system
- Same chunk size and overlap strategy
- Single embedding model fixed across all systems (selected via prior ablation; see `configs/embedding.yaml`)
- Each query run 5× and averaged; standard deviation reported alongside means
- ANN index parameters set to **vendor defaults** — no per-system tuning (fairness policy)

---

## Statistical Analysis

Paired Wilcoxon signed-rank tests with Bonferroni correction for multiple comparisons, applied per workload type across NDCG@10 scores.

---

## Repository Structure

```
empirical-rag-paradigm-benchmark/
├── data/               # Corpus, query sets, and ground-truth relevance judgments
├── src/
│   ├── ingestion/      # ArXiv loader, chunker, embedder
│   ├── databases/      # Client wrappers for pgvector, MongoDB, Qdrant
│   ├── benchmark/      # Runner, metrics, latency tracking, workload definitions
│   └── analysis/       # Statistical tests and visualization
├── notebooks/          # EDA, embedding selection, results analysis
├── results/            # Raw per-run JSON outputs and generated figures
├── configs/            # DB connection, benchmark, and embedding config
├── tests/              # Unit tests for metrics, clients, and chunker
└── docs/               # Proposal PDF and methodology notes
```

---

## Setup

### Prerequisites

- Python 3.12.0
- Supabase account (free tier sufficient)
- MongoDB Atlas account (free tier sufficient)
- Qdrant Cloud account (free tier sufficient)
- OpenAI API key

### Installation

```bash
git clone https://github.com/Sakhiur2022/empirical-rag-paradigm-benchmark.git && cd empirical-rag-paradigm-benchmark && code .

python -m venv empirical-rag-paradigm-benchmark
empirical-rag-paradigm-benchmark\Scripts\activate

pip install -r requirements.txt
cp .env.example .env             # Fill in connection strings and API keys
```

## Running the Benchmark

```bash
# 1. Download and chunk the corpus
python -m src.ingestion.arxiv_loader --config configs/benchmark.yaml

# 2. Generate and cache embeddings
python -m src.ingestion.embedder --config configs/embedding.yaml

# 3. Ingest into all three databases
python -m src.ingestion.arxiv_loader --ingest-all

# 4. Run the full benchmark across all workloads and systems
python -m src.benchmark.runner --config configs/benchmark.yaml

# 5. Run statistical analysis
python -m src.analysis.statistical_tests --results results/raw/
```

Results are written to `results/raw/` as JSON; figures are saved to `results/figures/`.

---

## Key References

- Lewis et al. (2020). Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks. _NeurIPS_. https://doi.org/10.48550/arXiv.2005.11401
- Thakur et al. (2021). BEIR: A Heterogeneous Benchmark for Zero-shot Evaluation of Information Retrieval Models. _NeurIPS Datasets and Benchmarks_. https://doi.org/10.48550/arXiv.2104.08663
- Muennighoff et al. (2022). MTEB: Massive Text Embedding Benchmark. https://doi.org/10.48550/arXiv.2210.07316
- Gao et al. (2023). Retrieval-Augmented Generation for Large Language Models: A Survey. https://doi.org/10.48550/arXiv.2312.10997

---

## License

MIT License. See [LICENSE](LICENSE) for details.

---

## Authors

| Name                 | Student ID | Role            |
| -------------------- | ---------- | --------------- |
| A.M.A Sakhiur Rahman | 2231507642 | Lead researcher |
| Sadman Jawad Suborno | 2233397642 | Co-researcher   |
