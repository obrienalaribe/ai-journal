---
date: 2026-03-02
source: reflect
issue: "#218"
tags: [architecture, patterns]
blog_worthy: false
---

# GC-on-Read: Enforce TTL at the Read Layer, Not a Separate Cron

## What Happened
An n8n error monitoring system had a documented 7-day TTL but no enforcement — `readErrors()` returned everything from disk regardless of age. The fix: filter stale entries on every read and write back only when entries were actually pruned.

## The Insight
For filesystem-backed ring buffers without a database, GC-on-read is the simplest TTL strategy. No cron, no background timer, no separate cleanup job. The read path is the cleanup path.

## By The Numbers
- 10 LOC added to `readErrors()`, 10-line E2E test added
- 0 build errors, 0 iterations, 7/7 E2E tests pass

## Pattern
GC-on-read with conditional write-back: filter → compare length → write only if pruned.
