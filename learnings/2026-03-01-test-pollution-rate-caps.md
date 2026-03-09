---
date: 2026-03-01
source: reflect
issue: "#194"
tags: [testing, debugging, patterns]
blog_worthy: false
---

# Stale Test Data Silently Triggers Rate Limits

## What Happened
E2E tests for content delivery queue were failing with 429 rate limit errors. Root cause: previous test runs left "posted" queue items that accumulated beyond the daily cap (5). Test cleanup only cancelled items in scheduled/failed/cancelled states — it couldn't touch posted items because no DELETE endpoint existed.

## The Insight
When E2E tests mutate shared state with irreversible transitions (posted, completed), cleanup must use hard deletes, not soft status changes. PATCH-to-cancelled is insufficient when the system counts records regardless of lifecycle stage.

## By The Numbers
- 25 existing tests, 2 failing, 5 missing -> 30 tests, 0 failing
- 20 LOC backend changes enabled the fix (DELETE endpoint + expire-stale endpoint)

## Pattern
Hard-delete cleanup for E2E test isolation over soft-status-change cleanup
