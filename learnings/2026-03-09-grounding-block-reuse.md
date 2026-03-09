---
date: 2026-03-09
source: reflect
issue: "#337"
tags: [architecture, patterns]
blog_worthy: false
---

# Reuse grounding helpers across generation endpoints — don't reinvent the prompt injection

## What Happened
LinkedIn post generation ignored pinned intelligence sources. Brief generation already had grounding via `buildGroundingReference` + `formatGroundingBlock`. Applied the same 15-line pattern to the LinkedIn handler instead of building a new abstraction.

## The Insight
When adding context injection to a second endpoint, reuse the existing helper functions with the same signature. The cost of a new abstraction layer (factory, strategy pattern) isn't justified until 3+ consumers exist.

## By The Numbers
- 15 lines added to schedule.js (grounding block injection)
- 0 new functions created — 100% reuse of existing helpers

## Pattern
Helper-function reuse over abstraction-layer creation for the second consumer
