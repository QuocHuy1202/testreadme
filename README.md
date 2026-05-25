# The Digital Contract Hub — Technical Solution Design & README

**Candidate:** Huynh Quoc Huy  
**Target Position:** AI Innovation Engineer (Fresher/Intern)  
**Project:** Problem 2 — Digital Contract Hub  
**Date:** May 2026  

---

## 1. Executive Summary & Capabilities
**The Digital Contract Hub** is an end-to-end intelligent document processing (IDP) platform designed to digitize, index, and instantly search physical and scanned legal contracts across OnPoint's Legal, Finance, and Operations teams. Built upon a **Hybrid Architecture**, the tool optimizes operational costs and latency by decoupling simple search routines from heavy semantic inquiries.

* **Deterministic Metadata Search:** An instantaneous, millisecond-latency search engine that filters scanned repositories by Partner Names, Signing Dates, or Clause Types using optimized regular expressions without invoking costly AI pipelines.
* **Semantic Q&A Chatbot (RAG):** A deep document understanding assistant leveraging a robust Hybrid Retrieval strategy (Dense Vectors + Sparse Keywords) coupled with a local Large Language Model (Qwen 2.5-7B) to interpret complex legal prompts and synthesize highly condensed, factual answers.

---

## 2. Achieved Performance vs. OnPoint Targets

| Evaluation Criterion | OnPoint Target | Achieved (PoC) | Target Status |
| :--- | :--- | :--- | :--- |
| **OCR / Parsing Accuracy on Key Fields** | > 99.00% | 93.56% | Pending Layout-Aware Tuning |
| **Retrieval Precision @ 3** | > 90.00% | 100.00% | **Target Met** |
| **Exact Clause & Page Source Citation** | Required (100%) | 100.00% | **Target Met** |
| **Clause Extraction Recall** | > 85.00% | 100.00% | **Target Met** |
| **Average End-to-End Latency** | N/A (Propose) | 8.07 seconds | **Operational** |

### Metric Insights & Analytical Explanations:
* **OCR Target (93.56% vs 99%):** The base `EasyOCR` model introduces character noise on certain complex formatted dense terms. Standard text cleaning patterns were applied to counteract this, but upgrading to layout-aware parsing is scheduled for production.
* **Retrieval, Citation, and Recall (100%):** The system achieved perfect technical fidelity during local validation due to an explicitly constrained context boundary, which is critically analyzed in Section 4.
* **Inference Latency (8.07s):** This includes heavy local token generation using a 7-Billion parameter model executing on a shared Cloud GPU (Tesla T4) without cloud API dependency costs.

---

## 3. Technical Architecture & System Design
The system follows a completely local pipeline, eliminating data compliance risks (NDAs) and ensuring zero external API cost overhead:
1. **Dual-Mode Data Ingestion:** The system reads native PDF text streams instantly using `pypdf`. If a blank or scanned page block is detected, it triggers an automated fallback to a GPU-accelerated `EasyOCR` engine.
2. **Legal-Bound Clause Chunking:** Instead of naive character-limit slicing which breaks legal coherence, the engine splits documents using standard legal markdown boundaries (e.g., regex splits matching `"Điều \d+"` / `"Clause \d+"`). This guarantees every chunk is an atomic, self-contained legal clause.
3. **Hybrid Indexing & Retrieval:** Merges keyword lookup (`BM25Retriever`) with deep contextual semantic matching (`BAAI/bge-m3` dense embeddings stored in a local `Chroma` vector database) to avoid vocabulary mismatching in dense legal documentation.
4. **Rigid Prompt Engineering & Quantized Local LLM:** Executes `Qwen2.5-7B-Instruct` compressed into 4-bit weights (NF4) via `BitsAndBytes`. A system prompt forces the LLM to output immediate, short legal facts and completely prohibits unanchored explanations or hallucinations.

---

## 4. Critical Evaluation Limitations & Scalability Roadmaps

### The "Small Context" Phenomenon
It is imperative to note that the near-perfect metrics achieved in this Proof of Concept (100% Retrieval Precision and Clause Recall) are directly highly correlated with the **miniature evaluation footprint**. The validation dataset consisted of a golden testing set of 5 highly structured legal queries applied against a single 4-page template contract. 

In this closed micro-environment, embedding collisions are mathematically negligible, semantic density is low, and the entire document text fits comfortably within a single retrieval window, making high scores easily achievable.

### Production-Scale Failure Modes (Moving to Thousands of Contracts)
Scaling this solution to an enterprise production database containing tens of thousands of diverse, multi-page commercial agreements will trigger clear performance degradation:
* **Semantic Collision and Dilution:** Commercial contracts share vast volumes of identical boilerplate legalese (e.g., standard confidentiality, severability, and force majeure clauses). In a massive vector space, dense embeddings will cluster together tightly, dropping Retrieval Precision@3 significantly as the vector store returns text blocks from entirely unrelated vendors.
* **Context Window Pollution & Noise:** As search results contain more noise, passing unrelated fragments into the LLM context prompt will exceed optimal token capacities, trigger model confusion, and dramatically inflate hallucination risks.
* **OCR Pipeline Congestion:** Processing thousands of incoming scanned multi-page folders sequentially via basic `EasyOCR` will form severe computational queues, making real-time user ingestion impossible.

### The Large-Scale Solution Architecture
To transition the Proof of Concept into an enterprise-grade system, the following architectural upgrades will be deployed:
* **Knowledge Graphs & Graph RAG:** Replace or augment flat vector databases with a Knowledge Graph (e.g., Neo4j). Relationships will be explicitly mapped: `(Company A) -[:SIGNED]-> (Contract X) -[:CONTAINS]-> (Clause 4: Confidentiality)`. This completely circumvents semantic vector collisions by making document boundaries deterministic and query filtering absolute.
* **Hierarchical Parent-Child Chunking:** Index highly granular, small text fragments (child chunks) to optimize precise retrieval matches, but fetch and pass the wider surrounding context block (parent chunk) to the LLM to guarantee thematic legal awareness.
* **Advanced Layout-Aware Document Parsers:** Swap out EasyOCR for multi-modal layout parsers like `LayoutLMv3` or `Surya OCR`. This retains precise physical column hierarchies and parses intricate complex legal data tables perfectly.
* **Domain-Specific Embedding Fine-Tuning:** Fine-tune the `bge-m3` embedding layer using contrastive learning on OnPoint's historical English-Vietnamese commercial agreements to dramatically increase spatial separation between highly similar clauses.

---

## 5. Agentic Coding Journey
Artificial Intelligence was extensively woven into the implementation lifecycle of this assignment, serving as an advanced pair-programmer:
* **Regex Optimization Chaining:** Crafting robust regular expressions to capture highly irregular Vietnamese date declarations and volatile business entity formats across raw OCR text blocks was accelerated by feeding string samples into an AI assistant, utilizing prompt chaining to iteratively stress-test the patterns.
* **UI Rapid Prototyping:** The clean dual-column web interface was instantly structured by utilizing AI to write modular, clean boilerplate `Gradio` components, shifting focus toward core algorithm tuning.
* **Deterministic Output Formatting:** AI was leveraged to refine the strict prompt format rules, adjusting token logit constraints and structuring clear few-shot templates to ensure Qwen consistently respects citation bounds.

---

## 6. Deployment & Replication Instructions
To execute the tool and replicate the technical dashboard results, follow these straightforward steps:
1. Open the provided notebook file `the-digital-contract-hub.ipynb` within Google Colab or Kaggle.
2. Ensure your environment's Accelerator runtime is set to a GPU configuration (e.g., **NVIDIA Tesla T4 GPU**).
3. Execute all cells sequentially. The initial phase will automatically fetch and configure core poppler binaries, language dictionaries, and 4-bit quantizations.
4. Upon running Phase 5, locate the terminal output displaying a public shared URL (e.g., `https://<identifier>.gradio.live`). Click this link to engage with the live interactive dashboard.
