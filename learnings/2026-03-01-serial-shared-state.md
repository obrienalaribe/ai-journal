---
date: 2026-03-01
source: reflect
issue: "#194"
tags: [testing, ci, patterns]
blog_worthy: false
---

# Parallel test blocks with shared server state cause silent cross-contamination

## What Happened
E2E tests in a single file had 7 `test.describe` blocks, each with `mode: "serial"` internally. But Playwright's `fullyParallel: true` ran different blocks on different workers simultaneously. A rate-cap test filled the server's daily post limit, causing an unrelated test block's claim to get 429. Fix: file-level serial mode + eliminating shared mutable variables across test groups.

## The Insight
`mode: "serial"` on a describe block only serializes tests *within* that block. When multiple serial blocks share server-side state (rate caps, global counters), the file itself needs file-level serial configuration. This is a Playwright footgun that's easy to miss.

## By The Numbers
- 2 failing tests → 0 (29 passed, 1 expected skip)
- 69 LOC changed in one file

## Pattern
File-level serial for shared server state — per-block serial is insufficient when blocks mutate global resources
