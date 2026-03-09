---
date: 2026-03-04
source: reflect
issue: "#236"
tags: [architecture, debugging, patterns]
blog_worthy: true
---

# LanceDB Virtual Columns: The Select Trap That Returns Zero Results

## What Happened
During a ground-up redesign of the Intelligence Engine's LanceDB search pipeline, including `_distance` and `_score` in `.select()` caused "No field named" errors — they're virtual columns auto-projected by the engine, not schema columns. After removing them, an overly aggressive `distanceRange(0, 0.75)` threshold filtered ALL vector results because MiniLM-L6-v2 produces distances of 0.8-1.3 for relevant content. Two invisible failures stacked: wrong select + wrong threshold = zero useful search results that still returned 200 OK.

## The Insight
When building on top of databases that auto-project computed columns (LanceDB, DuckDB, Elasticsearch), always profile actual value distributions before setting filter thresholds. A "working" search that returns empty results is harder to debug than one that throws errors.

## By The Numbers
- 294 embeddings indexed across 6 source types after v3 schema migration
- 14 → 16 E2E intelligence tests (2 quality guards added)
- 3 metadata fields promoted from JSON blob to dedicated scalar columns (source_url, source_name, item_title)

## Pattern
Profile-before-threshold: Always `node -e` or `curl` to check actual value distributions before hardcoding numeric thresholds in search pipelines.
