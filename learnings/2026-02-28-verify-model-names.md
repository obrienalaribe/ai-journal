---
date: 2026-02-28
source: reflect
issue: "#178"
tags: [debugging, patterns, architecture]
blog_worthy: false
---

# Never Trust Spec Model Names — Verify Against Running Code

## What Happened
Issue spec referenced "claude-sonnet-4-6" as the model name. I used `claude-sonnet-4-6-20250514` which returned 404 from the Anthropic API. The actual model constant in server.js was `claude-sonnet-4-20250514`. Caught by curl testing.

## The Insight
External API identifiers (model names, endpoint paths, parameter values) should be verified against existing working code, not trusted from spec documents. Specs are written by humans who may use approximate names.

## By The Numbers
- 1 curl test caught the bug before commit
- 1 line fix (model name) + 1 line fix (process.env propagation)

## Pattern
Cross-reference-first: when using any identifier that maps to an external API, grep the codebase for the canonical value before using it.
