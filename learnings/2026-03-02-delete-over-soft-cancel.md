---
date: 2026-03-02
source: reflect
issue: "#209"
tags: [testing, patterns, architecture]
blog_worthy: false
---

# Soft Cancellation Is Not Cleanup

## What Happened
E2E tests created 18+ queue items, ideas, and atoms in production data stores. Tests "cleaned up" by PATCHing queue items to `cancelled` status, but items persisted in JSON files indefinitely. n8n workflows could pick up test data and attempt LinkedIn posts.

## The Insight
Soft-delete (status change) and hard-delete (removal) serve different purposes. Test cleanup requires hard-delete to prevent data accumulation. If your test cleanup uses status transitions instead of actual removal, you're hiding mess, not cleaning it.

## By The Numbers
- 18 stale E2E queue items in production before fix
- 0 E2E artifacts remaining after 57 tests run with new cleanup

## Pattern
Hard-delete test data, soft-delete user data. Tests need `DELETE` endpoints, not just status-transition `PATCH` endpoints.
