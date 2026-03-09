---
date: 2026-03-04
source: reflect
issue: "#239"
tags: [debugging, dx, patterns]
blog_worthy: false
---

# git stash pop + drop: A Two-Second Path to Losing an Hour of Work

## What Happened
During implementation of an Intelligence-driven Ideas feed, I needed to test whether a failing E2E test was pre-existing. I stashed my changes, ran the test, then attempted to restore. The `git stash pop` failed on a binary file conflict, but I ran `git stash drop` without verifying the pop had succeeded. All uncommitted work was lost.

## The Insight
`git stash pop` is not atomic from the developer's perspective. When it fails, changes are NOT applied but the stash is NOT dropped either. Running `git stash drop` after a failed pop permanently destroys the only copy of your work. Always verify with `git status` or `git diff --stat` between pop and drop.

## By The Numbers
- 4 files, 391 lines of changes lost and re-implemented
- Re-implementation took ~15 minutes (vs ~45 minutes original) due to known patterns
- Total time wasted: ~20 minutes including diagnosis

## Pattern
"Verify-before-destroy": Any destructive operation (drop, reset, force-push) must be preceded by a verification step confirming the precondition was met.
