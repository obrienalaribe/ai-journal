---
date: 2026-03-01
source: reflect
issue: "#204"
tags: [patterns, architecture]
blog_worthy: false
---

# GC-on-read prevents unbounded JSON file growth

## What Happened
A JSON ring buffer for error logs accumulated 51 stale test entries because the TTL filter only applied in-memory — the file on disk never shrank. Added a conditional write-back (only when items actually removed) to garbage-collect expired entries during normal reads.

## The Insight
For file-backed data stores with TTL, filtering on read without writing back creates a memory leak on disk. The fix is 3 lines: compare filtered length to original, write back if different.

## By The Numbers
- 51 stale entries → 0 after fix (7-day TTL + GC)
- 3 lines of code for the GC write-back

## Pattern
GC-on-read: filter expired entries during read, persist cleaned data only when entries were actually removed (avoids write amplification on clean reads).
