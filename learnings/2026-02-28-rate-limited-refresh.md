---
date: 2026-02-28
source: reflect
issue: "#169"
tags: [architecture, patterns]
blog_worthy: false
---

# Rate-Limited On-Demand Refresh: The Three-Rule Pattern

## What Happened
Dashboard valuation data (MVRV, 200W MA) only refreshed nightly. Added a POST endpoint that runs the valuation script on-demand with a 5-minute in-memory rate limit. The key design decision: reset the rate-limit timestamp on failure so users aren't locked out by transient errors, and merge into the snapshot file (read→merge→write) rather than replacing it.

## The Insight
On-demand refresh endpoints that delegate to external scripts need three safety properties: rate-limit with cooldown, failure-aware cooldown reset, and merge-not-replace semantics for multi-field data files.

## By The Numbers
- 2 files changed, 108 insertions, 10 deletions
- 0 build errors, 0 type errors on first implementation

## Pattern
Rate-limited script delegation: `timestamp check → exec script → parse stdout → merge into file → respond`
