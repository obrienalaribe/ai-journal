---
date: 2026-03-05
source: reflect
issue: "#253"
tags: [architecture, debugging, patterns]
blog_worthy: true
---

# Validate SDK APIs Against Type Definitions, Not Agent Assumptions

## What Happened
An external critique identified 8 anti-patterns in our LanceDB intelligence engine. We validated each claim against the actual @lancedb/lancedb v0.26.2 type definitions before implementing. 4 claims were valid (mergeInsert upsert, batch embedding, native RRF hybrid, dummy row init). 4 claims were hallucinated (queryType("hybrid"), MultiMatchQuery doesn't exist, Index.bitmap() unstable, vector_rich needed). The implementation agent then hallucinated `_relevanceScore` as an RRF output field name -- it doesn't exist in the SDK. The fallback masked the bug.

## The Insight
When reviewing AI-generated code that references SDK-specific virtual column names or method signatures, the only source of truth is the type definition files in node_modules. Reading .d.ts files takes 2 minutes and prevents shipping code with hallucinated APIs. This applies equally to external critiques and to your own agent delegation outputs.

## By The Numbers
- 10 ingest functions refactored from O(N) sequential to O(N/8) batched
- 376 lines removed, 326 added (net -50 lines for same functionality)
- 4/8 critique claims falsified by reading actual .d.ts files
- 24/24 E2E tests pass after refactor

## Pattern
"Type-definition-first validation" -- before accepting any claim about an SDK's API surface (from humans, AI critiques, or agent outputs), grep the package's .d.ts files for the exact method/field name. If it's not there, it doesn't exist.
