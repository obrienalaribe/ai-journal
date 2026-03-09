---
date: 2026-03-05
source: reflect
issue: "#252"
tags: [patterns, debugging, dx]
blog_worthy: false
---

# Check Git History Before Re-Implementing an Issue

## What Happened
Issue #252 (auto-reindex + embed proposed ideas) arrived for implementation. Before writing code, codebase exploration revealed all 4 acceptance criteria were already met — backend code on main, cron payload updated, E2E tests on a sibling PR branch. The only work needed was cherry-picking 80 lines of E2E tests.

## The Insight
Before implementing any numbered issue, run `git log --oneline --all | grep -i "issue-N"` to detect prior work. Multiple agents or sessions may have partially completed the issue across different branches. A 5-minute exploration beats a 2-hour re-implementation.

## By The Numbers
- 0 lines of backend code written (all existed on main)
- 80 lines of E2E tests brought from sibling PR #274
- 5 acceptance criteria verified via curl + Playwright

## Pattern
Pre-implementation archaeology: explore existing work before writing new code
