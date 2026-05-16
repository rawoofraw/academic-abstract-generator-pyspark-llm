# Distributed Academic Abstract Generator: PySpark & LLM Pipeline

This repository contains a two-stage enterprise-level machine learning pipeline designed to process large-scale, unstructured academic data and use it to instruction-tune a lightweight Large Language Model.

## Project Architecture



The project is split into two discrete, production-ready phases to optimize resource utilization and prevent out-of-memory errors:

1. **Phase 1: Distributed Data Engine (PySpark):** Ingests raw, multi-gigabyte arXiv JSON metadata, applies complex regular expressions via User Defined Functions (UDFs) to strip LaTeX formatting/citation noise, filters by research domain, and exports partitioned Parquet files.
2. **Phase 2: Instruction-Tuning (Unsloth / QLoRA):** Loads the structured Parquet files, tokenizes the text into numeric tensors, and fine-tunes a `Llama-3.2-3B-Instruct` model to generate technical abstracts based entirely on a paper's title.

## Phase 1: Data Engineering Highlights
* **Resource Optimization:** Configured PySpark to run in local cluster mode using custom memory allocation (`24g` driver memory) to process heavy text datasets efficiently on commodity cloud hardware.
* **Noise Reduction:** Applied distributed regex cleaning to wipe out embedding-shattering noise like embedded math syntax (`\begin{equation}`) and bracketed citations (`[1, 2]`).
* **Storage Optimization:** Implemented `partitionBy` structures to organize data into folder lakes based on primary academic categories, significantly lowering I/O latency for downstream modeling.
