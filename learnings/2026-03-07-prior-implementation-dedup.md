---
date: 2026-03-07
source: reflect
issue: "#302"
tags: [patterns, ci]
blog_worthy: false
---

# Check for prior partial implementations before coding from spec

## What Happened
Issue #302 spec said "add qa-report.md check, same as current after-*.png hard-block." On reading the target file, a prior agent had already added a basic evidence gate (step 6). Adding a second comprehensive gate (step 2b) created a duplicate `const` declaration that would crash at runtime.

## The Insight
When multiple agents work on related issues sequentially, later specs can be stale about what the file already contains. Always read the target file BEFORE trusting the spec's description of its current state.

## By The Numbers
- 1 duplicate `const` caught before commit
- 1 unused import removed during self-review

## Pattern
"Read-before-spec" — treat the spec as intent, the file as truth.
