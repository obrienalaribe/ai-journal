---
date: 2026-03-08
source: reflect
issue: "#333"
tags: [architecture, patterns]
blog_worthy: false
---

# One Field, Two Namespaces — A Silent Data Corruption Pattern

## What Happened
A `source_ids` field on content ideas was repurposed to store LanceDB document IDs alongside its original SRC-xxx content source IDs. Downstream consumers (correlate endpoint, brief generation) matched against the wrong namespace — silently returning empty results. The fix was adding a dedicated `pinned_intelligence_ids` field and migrating the data.

## The Insight
When two ID namespaces share a single array field, every consumer silently breaks for the wrong namespace. The fix is always a new field — never "make consumers handle both." Namespace collision is a schema bug, not a consumer bug.

## By The Numbers
- 12 LanceDB IDs moved from `source_ids` to `pinned_intelligence_ids` on idea-007
- 11 files touched across backend, frontend, types, tests, and data

## Pattern
Separate-namespace-per-field: each array field must contain IDs from exactly one namespace. If a feature needs IDs from a different namespace, add a new field.
