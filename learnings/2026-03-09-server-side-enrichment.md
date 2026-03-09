---
date: 2026-03-09
source: reflect
issue: "#338"
tags: [architecture, patterns, testing]
blog_worthy: false
---

# Server-side enrichment beats client-side orchestration for create flows

## What Happened
A Search-to-Create-Idea flow was built without post-creation enrichment: ideas got hardcoded pillar, zero score, empty summary. Rather than having the frontend orchestrate multiple API calls after creation (recalculate score, generate summary, infer pillar), the enrichment was moved into the POST handler itself using existing utilities (fetchByIds, recalculateIdeaScore, computeCOS).

## The Insight
When a create endpoint can access all enrichment data server-side, doing it inline in the handler (with try/catch fallbacks) produces complete objects in one round-trip. This simplifies E2E testing, eliminates race conditions, and keeps the frontend thin.

## By The Numbers
- 4 acceptance criteria satisfied in 1 POST response vs 3 sequential API calls
- 94 lines of backend enrichment logic added, 9 lines of frontend changes

## Pattern
Inline enrichment with graceful degradation: try each enrichment step in a try/catch, fall back to defaults, return the best-effort result in one response.
