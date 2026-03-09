---
date: 2026-03-01
source: reflect
issue: "#186"
tags: [architecture, patterns]
blog_worthy: false
---

# Ring buffers beat append-only logs for UI-facing event feeds

## What Happened
The content pipeline's "Last Pipeline Run" section only tracked video-oriented skill timings. LinkedIn generation, queue operations, and approvals were invisible. Added a JSON ring buffer (50 events max) with typed events logged from 4 existing endpoints, served via a new GET endpoint and rendered as an activity feed.

## The Insight
When an event log exists primarily to power a UI feed (not for audit/compliance), a capped JSON array with atomic writes is simpler and more predictable than append-only JSONL. The consumer reads one file, gets the latest N events, no parsing or streaming needed.

## By The Numbers
- 4 files changed, ~70 net lines added
- 5 event types tracked (generate, queue_posted, queue_failed, approve, status_change)
- 1 TypeScript error caught by `tsc --noEmit` (strict array access)

## Pattern
JSON ring buffer for UI event feeds — atomic read/write, fixed size, newest-last ordering.
