---
date: 2026-02-26
source: reflect
issue: "#79"
tags: [patterns, architecture]
blog_worthy: false
---

# Extract Transform Helpers Before Submitting PRs

## What Happened
A news headlines endpoint had 3-tier fallback logic (fresh pillars, keyword-filtered, stale pillars). Each tier had identical sort-slice-map pipelines — 3x ~10 lines of duplicated transforms. The PR was approved by the automated reviewer but OB flagged the DRY violation during human review, requiring a revision cycle.

## The Insight
When a function has multiple return paths with the same data transformation, extract the transform into a named helper before the first review. The cost of a revision cycle (re-review, re-test, re-deploy) always exceeds the cost of 5 minutes of upfront extraction.

## By The Numbers
- 35 lines of duplicated transforms reduced to 3 one-liner calls
- 1 review cycle saved if caught before initial submission

## Pattern
Extract-before-submit: any time you write the same transform pipeline twice, stop and extract immediately.
