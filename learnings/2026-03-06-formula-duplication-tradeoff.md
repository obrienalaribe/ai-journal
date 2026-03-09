---
date: 2026-03-06
source: reflect
issue: "#283"
tags: [architecture, patterns, dead-code]
blog_worthy: false
---

# When to duplicate a formula vs extract it

## What Happened
Added a COS (Content Opportunity Score) backfill feature that reuses a scoring formula from the intelligence module. The formula was inline in a route handler, not a standalone function. Rather than refactoring the source module, I duplicated the formula into a local helper with a "mirrors X" comment and flagged it for later extraction.

## The Insight
For small, stable formulas (< 25 lines) with exactly 2 consumers, duplicating with a tracking comment is faster than refactoring the source module in the same PR. The key is making the duplication discoverable: a `mirrors` comment + a Living Rule with a grep guard ensures the next editor knows both copies must stay in sync.

## By The Numbers
- 3 files changed, ~130 lines added (84 backend, 44 frontend)
- 0 build errors, 0 iterations needed

## Pattern
Duplicate-with-tracking: copy formula + add `mirrors file:function` comment + Living Rule grep guard. Schedule `/refactor` to extract.
