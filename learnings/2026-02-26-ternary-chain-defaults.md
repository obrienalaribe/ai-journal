---
date: 2026-02-26
source: reflect
issue: "#83"
tags: [patterns, debugging]
blog_worthy: false
---

# Ternary chain fallback traps when adding size variants

## What Happened
Added an `xs` size variant to a ScoreCircle component that used a ternary chain for size mapping. The original default (36px for undefined size) silently shifted to 20px (xs) because the new variant was placed as the final fallback. Caught during self-review before commit.

## The Insight
When extending ternary chains with new variants, the fallback position determines the implicit default. New variants should be inserted mid-chain, never as the terminal fallback — or better yet, use an object lookup with an explicit default key.

## By The Numbers
- 2 files changed, 24 insertions, 21 deletions
- 1 near-regression caught in self-review (would have broken IdeasTab ScoreCircle)

## Pattern
Object-lookup-over-ternary for multi-variant sizing: `const s = SIZES[size ?? "md"]` prevents implicit default drift.
