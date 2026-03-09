---
date: 2026-02-26
source: reflect
issue: "#84"
tags: [patterns, dead-code, dx]
blog_worthy: false
---

# Duplicate Components Drift Apart — Consolidate Early

## What Happened
Two ScoreCircle components existed in parallel — one in shared.tsx (square, size variants) and one in IdeasTab.tsx (circular, oversized at 44px). Over time the local copy drifted from the shared design system, creating a visual bug where the score circle dominated the card header.

## The Insight
When you copy a component instead of importing the shared one, the copies inevitably diverge. The fix is always consolidation with a variant prop — not tweaking the copy. The earlier you consolidate, the less drift accumulates.

## By The Numbers
- 2 files changed, 24 insertions, 32 deletions (net -8 lines)
- Score circle: 60px total height (44px + /10 text) reduced to 32px (inline /10)

## Pattern
Variant-prop consolidation: extend shared components with `shape`/`variant` props rather than maintaining parallel copies.
