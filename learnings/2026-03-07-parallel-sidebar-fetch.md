---
date: 2026-03-07
source: reflect
issue: "#295"
tags: [patterns, hooks, performance]
blog_worthy: false
---

# Parallel sidebar data needs a unified AbortController, not multiple useApi hooks

## What Happened
A creative workspace tab needed 4 parallel API fetches (brief markdown, semantic correlations, stories, photos) to populate a two-panel layout. Using separate `useApi()` hooks would have created 4 independent refresh cycles with no shared abort logic. Instead, a single `useEffect` with one `AbortController` and `Promise.all` with per-fetch `.catch(() => default)` kept the fetches atomic and cleanly cancelable.

## The Insight
When a component needs N parallel fetches that all depend on the same trigger (selected item ID), a single effect with Promise.all is simpler and more correct than N independent hooks. The key pattern: each individual fetch catches its own errors with a sensible default, so one failure doesn't block the others.

## By The Numbers
- 4 parallel fetches consolidated into 1 effect with 1 AbortController
- 890 -> 1211 lines (net +320 for full inspiration sidebar, lightbox, research chips)

## Pattern
Single-AbortController parallel fetch with per-promise graceful fallback
