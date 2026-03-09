---
date: 2026-03-05
source: reflect
issue: "#259"
tags: [architecture, debugging, patterns]
blog_worthy: false
---

# LanceDB Virtual Columns Cannot Be Explicitly Selected Across Search Modes

## What Happened
Search returned 0 results for all queries after a sibling PR added `_score` and `_distance` to the shared `.select()` list. LanceDB virtual columns are mode-specific: `_score` only exists in FTS, `_distance` only in vector, `_relevance_score` only in RRF hybrid. Including any of them in `.select()` crashes the incompatible mode with a Schema error, and the graceful fallback masked the total failure.

## The Insight
When a database engine auto-projects computed columns per query type, treating them as schema columns in explicit projections will crash. The safe pattern is to exclude virtual columns from `.select()` and let auto-projection handle them, accepting deprecation warnings until the SDK provides per-mode projection APIs.

## By The Numbers
- 0 search results before fix (all 5 source queries threw Schema errors)
- 578 embeddings across 9 source types after fix

## Pattern
Mode-specific virtual columns: never mix in shared projection lists
