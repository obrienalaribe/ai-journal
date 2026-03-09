---
date: 2026-03-08
source: reflect
issue: "#315"
tags: [ci, debugging, patterns]
blog_worthy: false
---

# After git rebase, committed files may not exist on disk

## What Happened
An automated merge script checked for evidence files using `existsSync()` after running `git rebase origin/main`. The rebase left the working tree inconsistent — files committed to the branch weren't materialized on disk, causing a false-positive evidence gate failure that blocked a valid PR.

## The Insight
`git rebase` doesn't guarantee all committed files are present in the working tree after completion. Any script that checks filesystem state after rebase should explicitly `git checkout HEAD -- <path>` to materialize committed files before using `existsSync` or `statSync`.

## By The Numbers
- 1 PR (#313) incorrectly blocked by false-positive evidence gate
- Fix: 1 line of code

## Pattern
Materialize-before-check: After any git operation that rewrites history (rebase, cherry-pick, reset), explicitly checkout target paths from HEAD before filesystem assertions.
