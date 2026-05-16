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

## Phase 2: Instruction-Tuning Highlights
* **Efficient Compute Utilization:** Leveraged the `Unsloth` framework to fine-tune a `Llama-3.2-3B-Instruct` architecture, cutting VRAM overhead significantly and speeding up execution on a single NVIDIA P100 GPU.
* **Pre-Tokenized Optimization:** Implemented manual tokenization mapping and dropped redundant string features to optimize training data streams, resolving text-collator sequence length issues prior to gradient optimization.
* **Hyperparameters Configured:**
  * **Quantization:** 4-bit NormalFloat (`BitsAndBytes`)
  * **LoRA Rank ($r$):** 16 / **Alpha ($\alpha$):** 16
  * **Optimizer:** 8-bit AdamW
  * **Learning Rate:** 2e-4 with a linear scheduler

## Sample Inference Evaluation
To verify model learning, a zero-shot inference test was run on a completely synthetic title. The fine-tuned adapters successfully captured academic syntax, structure, and domain vocabulary:

### Input Title:
`Generative Approaches for Biomedical Data Augmentation Using VAEs and DDPM`

### Model Output:
"In this paper, we propose a novel generative framework for biomedical data augmentation to address the challenge of limited annotated medical datasets. By leveraging Variational Autoencoders (VAEs) alongside Denoising Diffusion Probabilistic Models (DDPM), our architecture effectively models complex physiological distributions and generates high-fidelity synthetic samples..."
