---
date: 2026-03-09
source: reflect
issue: "#337"
tags: [ci, patterns, debugging]
blog_worthy: false
---

# When Your Sibling PR Deletes the Function You Need

## What Happened
PR #341 added LinkedIn grounding using `fetchByIds()` from intelligence.js. Two sibling PRs (fix/issue-335, fix/issue-338) independently refactored the same file. fix/issue-338 removed `fetchByIds()` as dead code within its scope. Auto-merge failed because the rebased code lost the function silently.

## The Insight
In multi-agent parallel workflows, function "dead code" is only dead relative to one PR's scope. Cross-PR dependency analysis before merge would prevent this class of failure.

## By The Numbers
- 3 files changed, 46 lines added
- 2 sibling PRs required rebase resolution
- 1 function re-added that was valid but removed by sibling refactor

## Pattern
Post-rebase export verification: after every rebase on sibling branches, grep for all exports your code depends on to confirm they survived.
