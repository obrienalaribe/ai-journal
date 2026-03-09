---
date: 2026-03-02
source: reflect
issue: "#219"
tags: [testing, patterns]
blog_worthy: false
---

# E2E Tests Must Create Their Own Filesystem Fixtures

## What Happened
An E2E test validating photo file detection assumed a `.gitkeep` file existed in the data directory. In the main repo it did, but the git worktree had no `data/` directory. Test failed on first run, fixed by creating the file in test setup with `mkdirSync(recursive)` + `writeFileSync`.

## The Insight
Git worktrees, CI environments, and fresh clones don't share filesystem state outside the repo. E2E tests depending on data-directory files must be self-contained — create fixtures in setup, not assume they exist.

## By The Numbers
- 33 → 34 content-delivery E2E tests (4 new R3 fallback tests)
- 1 iteration saved by the fix (would have failed in CI too)

## Pattern
Self-contained test fixtures over environment-dependent assumptions
