---
date: 2026-03-08
source: reflect
issue: "#318"
tags: [architecture, refactor, patterns]
blog_worthy: false
---

# When splitting large files, shared helpers beat cross-imports

## What Happened
A 3976-line Express route file needed splitting into 3 modules. Three architectures were considered: cross-imports between route files, duplicated constants, or a shared helpers module. The shared module eliminated circular dependency risk and produced a clean acyclic dependency graph.

## The Insight
When 3+ consumers need the same functions, a shared helpers module is always the right first move. Don't debate cross-imports vs. duplication — the decision tree is trivial. Shared module avoids cycles, centralizes changes, and each consumer imports from exactly one place.

## By The Numbers
- 3976 lines → 4 files (1099 + 723 + 217 + 1511)
- 72 route handlers verified via curl, 0 regressions
- 15 min wasted on architecture deliberation (should have been 2 min)

## Pattern
Shared-helpers-module pattern: extract constants + pure functions into `*-helpers.js`, have all route modules import from it. Re-export upstream deps for single-import convenience.
