---
date: 2026-03-07
source: reflect
issue: "#282"
tags: [architecture, patterns]
blog_worthy: false
---

# Non-fatal evaluation gates keep pipelines robust

## What Happened
Added an LLM evaluation gate after script generation: Haiku scores the draft on 4 dimensions, optionally triggers a single regeneration pass with critique injected, and persists the higher-scoring version. The key design decision was making evaluation failure non-fatal — the brief is written to disk before the eval call, so Haiku outages or parse failures don't block the pipeline.

## The Insight
When adding quality gates to generative pipelines, always write the artifact first and evaluate second. Non-fatal eval gates add quality signal without introducing fragility. The regeneration pass uses a "pick higher" strategy rather than always trusting the second attempt.

## By The Numbers
- 3 files changed, +382 lines across backend eval + frontend score card
- 0 build errors on first attempt (existing Anthropic fetch pattern reused)

## Pattern
Non-fatal eval gate: generate -> persist -> evaluate -> conditional regen -> persist score
