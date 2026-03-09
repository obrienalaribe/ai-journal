---
date: 2026-03-08
source: reflect
issue: "#308"
tags: [architecture, patterns, performance]
blog_worthy: false
---

# Write the placeholder before starting the expensive job

## What Happened
A DeFi data endpoint blocked for 30-120s while running 14 parallel API scripts to generate today's snapshot. E2E tests with 5s timeouts failed because the GET handler awaited the full generation. Fix: write a minimal placeholder file synchronously, then run the full generation in background.

## The Insight
When an API listing endpoint triggers expensive background generation, write the minimal artifact first (so the listing is complete), then enrich it asynchronously. Callers get instant responses; the background job overwrites the placeholder with real data.

## By The Numbers
- Response time: 30-120s → <1s (placeholder returns immediately)
- Test pass rate: 0/1 → 1/1 for the affected test

## Pattern
Placeholder-before-generation: synchronous write of minimal file → fire-and-forget background enrichment → overwrite on completion
