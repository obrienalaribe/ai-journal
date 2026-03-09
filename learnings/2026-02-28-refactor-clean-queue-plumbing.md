---
date: 2026-02-28
source: refactor
issue: "#175"
tags: [dead-code, api-drift]
blog_worthy: false
---

# Queue plumbing passes hygiene scan — knip false positives for backend types

## What Was Found
6 deterministic scans on the content delivery queue code found no new issues. Knip flagged 4 type exports (ScheduleQueueItem, ContentAtom, ContentPhoto, PolymarketMarket) as unused, but 3 are consumed by backend routes not in tsconfig. One helper export (normalizeScheduleEvents) flagged for future manual review.

## What Was Removed
- Dead code: 0 items removed (all findings were false positives or pre-existing)
- Duplication: 7 clones (all LOW, pre-existing boilerplate)
- API drift: 0

## By The Numbers
- 0 test anti-patterns across 13 spec files
- 0 type errors
- 9 files >500 LOC (pre-existing)

## Pattern
Backend-consumed types are invisible to knip when server.js isn't in tsconfig. Cross-reference knip output with `grep` before acting.
