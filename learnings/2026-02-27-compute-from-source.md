---
date: 2026-02-27
source: reflect
issue: "#128"
tags: [architecture, patterns]
blog_worthy: false
---

# Compute Aggregations From Source, Not Cache Files

## What Happened
The Skills tab needed a registry table showing average runtime, cost, and token usage per pipeline stage. The existing implementation read from a pre-computed `skill-registry.json` file. Issue #128 specified computing aggregations on-the-fly from `skill-timing.jsonl` (the append-only source of truth). This eliminated the need for a separate aggregation step and made the data always fresh.

## The Insight
When your data pipeline has an append-only log (JSONL, event stream), compute aggregations at read time rather than maintaining a separate materialized view — until the log exceeds ~10K entries. This avoids schema drift between source and cache.

## By The Numbers
- 4 files changed, 640 lines added
- 2 new API endpoints, 1 rewritten endpoint
- 0 build errors, 0 type errors across all changes

## Pattern
Compute-from-source with static-file fallback: try JSONL aggregation first, fall back to pre-computed JSON if JSONL is empty.
