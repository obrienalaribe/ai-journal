---
date: 2026-03-09
source: reflect
issue: "#337"
tags: [testing, debugging, ci]
blog_worthy: false
---

# E2E tests that seed data via filesystem fail in worktrees

## What Happened
A brief score test wrote fixture data to `process.cwd()/../data/` to seed the server, but the Playwright tests run in a git worktree while the server runs from the main repo. The CWD-relative path doesn't match the server's DATA_DIR, so the API returns null. Four E2E test failures were blocking auto-merge for an unrelated PR.

## The Insight
When E2E tests seed data for a server, they must write to the server's actual data directory — not the test runner's CWD. Use API-based discovery (query server for existing data) instead of filesystem seeding for portability across worktrees and CI environments.

## By The Numbers
- 4 → 0 E2E failures after fixing path mismatch + 3 other test bugs
- 67 passed, 1 pre-existing flaky, 4 skipped

## Pattern
API-discovery seeding over filesystem-direct seeding for E2E test portability
