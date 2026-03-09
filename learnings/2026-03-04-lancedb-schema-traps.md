---
date: 2026-03-04
source: reflect
issue: "#236"
tags: [architecture, debugging, patterns]
blog_worthy: false
---

# LanceDB Schema Inference Has Silent Traps — Name Columns and Seed Carefully

## What Happened
Implemented a vector search layer using LanceDB (embedded) with local Xenova embeddings. Hit two schema-related failures: (1) `null` values in seed rows caused "Failed to infer data type" errors because LanceDB can't infer String from null, and (2) naming the embedding column `vec` instead of `vector` caused "No vector column found" on search because LanceDB auto-detects only `vector` by convention.

## The Insight
Embedded databases with schema inference (LanceDB, DuckDB) fail silently on edge cases that traditional schema-defined DBs catch at migration time. When using schema-on-write, always seed with fully-typed placeholder values and follow the library's naming conventions exactly.

## By The Numbers
- 3 files updated per schema fix (intelligence.js, embed-item.mjs, embed-all.mjs)
- 2 iterations wasted on column naming, 1 on null inference

## Pattern
Convention-over-configuration naming: LanceDB `vector`, Elasticsearch `embedding`, Pinecone `values` — each has a magic column name.
