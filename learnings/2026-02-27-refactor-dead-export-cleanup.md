---
date: 2026-02-27
source: refactor
issue: "#125"
tags: [dead-code, patterns]
blog_worthy: false
---

# Deprecated wrapper exports accumulate silently when replacements skip cleanup

## What Was Found
After replacing QualityCheckmarks with QualityGateBars + QualityDonut, the old component remained as a deprecated export with zero consumers. Two utility functions (pillarLabel, weightedScore) were exported but only used internally in the same file. Knip detected all 3 in a 10-second scan.

## What Was Removed
- 1 dead component export (QualityCheckmarks deprecated wrapper)
- 2 unnecessary exports de-scoped to file-private (pillarLabel, weightedScore)

## By The Numbers
- 3 unused exports removed in 3 edits
- tsc + build clean after each fix
- 8 pre-existing code clones flagged (all LOW, not addressed)

## Pattern
Run knip after every component replacement to catch deprecated wrappers before they fossilize
