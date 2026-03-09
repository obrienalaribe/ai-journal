---
date: 2026-02-28
source: reflect
issue: "#179"
tags: [architecture, patterns]
blog_worthy: false
---

# Design photo metadata pipelines with graceful fallback, not error paths

## What Happened
Added photo selection to LinkedIn content generation. The key design decision was making photos entirely optional at every stage: generation prompt adapts, atom fields nullable, claim response includes `photo_valid` boolean, n8n workflow branches on photo availability.

## The Insight
When adding optional enrichment to an existing pipeline, design the fallback as the primary path. The enrichment (photos) enhances but never blocks. Every consumer checks availability independently rather than trusting upstream state.

## By The Numbers
- 5 files modified, ~105 LOC backend + ~150 LOC frontend
- 0 build errors, 0 test regressions, 3 new E2E tests

## Pattern
"Graceful enrichment chains" — each stage independently validates optional data availability rather than trusting upstream state assertions.
