
````md
# Multimodal Fashion & Context Retrieval System

This repository contains an intelligent multimodal search engine that retrieves fashion images based on **natural language descriptions** involving clothing attributes, colors, environments, and style.

The goal is to move beyond simple keyword or vanilla CLIP-based retrieval and design a system that **works better for fashion-specific and compositional queries**, such as:

> “A red tie and a white shirt in a formal setting”

---

## 🚀 Project Motivation

Fashion retrieval is inherently **multimodal and compositional**:
- Multiple garments appear in one image
- Attributes like color, clothing type, and setting interact
- Queries are often descriptive, not categorical

While models like CLIP provide a strong zero-shot baseline, they struggle with:
- Fine-grained fashion attributes
- Attribute binding (e.g., which color belongs to which garment)
- Contextual cues like environment or vibe

This project explores a **CLIP-based system enhanced with attribute-aware reasoning** to improve retrieval quality for fashion queries.

---

## 🧠 High-Level Architecture

```text
┌────────────────┐
│ Image Dataset  │
└───────┬────────┘
        │
        ▼
┌────────────────────┐
│ CLIP Image Encoder │
└───────┬────────────┘
        │
        ▼
┌────────────────────┐
│ FAISS Vector Index │
└───────┬────────────┘
        │
        ▼
┌──────────────────────────┐
│  Top-K Candidate Images  │
└───────┬──────────────────┘
        │
        ▼
┌──────────────────────────┐
│ Compositional Reranking  │
│ (color / clothing / env) │
└─────────────┬────────────┘
              │
              ▼
        Final Ranked Results
````

---

## 📂 Repository Structure

```
.
├── indexer/
│   ├── feature_extractor.py   # CLIP image embedding logic
│   ├── build_index.py         # FAISS index construction
│
├── retriever/
│   ├── query.py               # Query-time retrieval pipeline
│   ├── query_parser.py        # Attribute extraction from text
│
├── evaluation/
│   └── metrics.py             # Precision@K computation
│
├── data/
│   ├── images/                # Fashion images
│   └── annotations/           # Optional Fashionpedia metadata
│
└── README.md
```

---

## 🗃 Dataset

A subset of the **Fashionpedia dataset** (≈ 500–1,000 images) is used.

### Dataset Coverage

* **Environments**: office interiors, urban streets, parks, home settings
* **Clothing Types**: formal wear, casual wear, outerwear
* **Colors**: wide variety of garment colors

Fashionpedia is particularly suitable due to its **fine-grained clothing annotations**, even though the core system remains zero-shot.

---

## 🔧 Indexing Pipeline (Part A)

The indexing pipeline converts raw images into a searchable vector database.

### Steps

1. Load images from disk
2. Encode images using CLIP’s image encoder
3. L2-normalize embeddings
4. Store embeddings in a FAISS index
5. Persist metadata (image path, optional attributes)

### Why FAISS?

* Simple and reliable
* Designed for large-scale similarity search
* Scales to millions of vectors
* Avoids unnecessary engineering overhead

```bash
python indexer/build_index.py
```

---

##  Retrieval Pipeline (Part B)

The retrieval pipeline accepts a **natural language query** and returns the top-K most relevant images.

### Steps

1. Encode query using CLIP text encoder
2. Retrieve top-K candidates from FAISS
3. Parse query into attributes:

   * Colors
   * Clothing types
   * Context / environment
4. Apply attribute-aware reranking
5. Return final ranked results

```bash
python retriever/query.py \
  --query "Professional business attire inside a modern office" \
  --top_k 5
```

---

## Compositional & Attribute-Aware Reasoning

### Why vanilla CLIP is not enough

CLIP often fails on:

* “Red shirt with blue pants” vs “Blue shirt with red pants”
* Queries involving multiple garments
* Context-heavy fashion descriptions

### Improvement Strategy

* Parse query into multiple semantic components
* Penalize retrieved images that violate attribute constraints
* Encourage alignment across **color + clothing + context**

This improves precision for **fashion-specific compositional queries** without retraining CLIP.

---

##  Evaluation Strategy

### Challenge

Fashion datasets do not provide explicit relevance labels for **free-form natural language queries**.

### Solution

Evaluation uses **proxy relevance signals**:

* Precision@K
* Keyword overlap between query attributes and image metadata (filename / annotations)

> This is an approximation, but sufficient for comparing retrieval quality and validating improvements over baseline CLIP retrieval.

---

##  Example Evaluation Queries

* **Attribute-specific**
  “A person in a bright yellow raincoat”

* **Contextual**
  “Professional business attire inside a modern office”

* **Complex semantic**
  “Someone wearing a blue shirt sitting on a park bench”

* **Style inference**
  “Casual weekend outfit for a city walk”

* **Compositional**
  “A red tie and a white shirt in a formal setting”


---

##  Summary

This project demonstrates a **thoughtful ML-first approach** to fashion retrieval:

* Strong zero-shot baseline
* Explicit handling of compositional queries
* Scalable design
* Clear understanding of limitations and tradeoffs

It prioritizes **model reasoning and retrieval quality** over unnecessary engineering complexity.

```

