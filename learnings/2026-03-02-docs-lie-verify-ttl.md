---
date: 2026-03-02
source: reflect
issue: "#204"
tags: [testing, data, debugging]
blog_worthy: false
---

# Documentation Claimed TTL Existed — It Didn't

## What Happened
Auto-memory stated "7-day TTL + GC-on-read" for the n8n error store, but the actual `readErrors()` function was a plain JSON read with no TTL filter. 51 stale E2E test entries accumulated in production, making the Activity Feed show false errors.

## The Insight
Documentation that describes aspirational behavior instead of actual behavior is worse than no documentation — it prevents investigation by making the reader think the problem is already solved.

## By The Numbers
- 51 stale entries → 0 after GC-on-read implemented
- 10 lines of server code + 30 lines of test changes

## Pattern
Trust-but-verify: When docs claim a data lifecycle feature (TTL, GC, expiry), grep the implementation to confirm it exists before relying on it.
