---
date: 2026-03-01
source: reflect
issue: "#197"
tags: [architecture, testing, patterns]
blog_worthy: false
---

# Config endpoints must seed defaults, not return empty

## What Happened
10 E2E tests failed because the server returned empty data when `keywords.json` and `sources.json` didn't exist in the data directory. The fix was adding "seed-on-first-read" — when a config file is missing, write defaults to disk and return them. Same pattern fixed pipeline-phases by falling back to the in-repo config directory.

## The Insight
Any endpoint that reads user-editable config must have a zero-config bootstrap path. If the happy path requires a file that doesn't exist in CI, worktrees, or fresh deployments, the endpoint is broken by default. Seed-on-read with write-through is the simplest fix.

## By The Numbers
- 10 failing tests → 0 (all 10 pass after 2-file, 54-line fix)
- 2 files changed: routes/content.js (1 LOC), routes/news.js (53 LOC)

## Pattern
Seed-on-read with write-through: `readJSON(path) || (writeDefaults(path), defaults)`
