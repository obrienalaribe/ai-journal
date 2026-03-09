---
date: 2026-03-05
source: reflect
issue: "#252"
tags: [testing, patterns]
blog_worthy: false
---

# Conditional E2E Tests Give False Confidence

## What Happened
An E2E test for LanceDB ingestion used `if (data.byType.proposed_idea !== undefined) { expect(...) }` — silently passing when the feature wasn't working. The fix: seed fixture data, trigger the pipeline, and assert unconditionally.

## The Insight
Tests that guard assertions behind data-existence checks never fail — they just skip. If a feature matters enough to test, seed the precondition in the test setup so the assertion always executes.

## By The Numbers
- 3 new unconditional tests added (seed + stats + correlate)
- Previous conditional test: 0 failures in 8 days despite stale index

## Pattern
Fixture-seeded serial tests: create → trigger → poll → assert → cleanup
