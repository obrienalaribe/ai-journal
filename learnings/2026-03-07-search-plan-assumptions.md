---
date: 2026-03-07
source: reflect
issue: "#281"
tags: [architecture, patterns, debugging]
blog_worthy: false
---

# Fixed Search Plans Hide Source Types You Expect To Find

## What Happened
Building a brief-generation endpoint that needed voice reference chunks from a vector database. The spec assumed the existing search API could return any source type, but the search function uses a fixed 5-source plan that excludes the needed type. Discovered this through grep before building, avoiding a broken implementation.

## The Insight
When a search system uses a curated routing plan (not a flat index scan), new source types are invisible until explicitly added to the plan. Always verify the search plan's source list before assuming a new type is queryable.

## By The Numbers
- 0 wasted iterations (caught during research, not trial-and-error)
- 1 new exported function (`searchByType`) added as clean workaround

## Pattern
Verify-before-assume: grep the search plan's source filters before designing data flows that depend on search results containing a specific type.
