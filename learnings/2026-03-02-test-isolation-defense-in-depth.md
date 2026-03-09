---
date: 2026-03-02
source: reflect
issue: "#209"
tags: [testing, architecture, patterns]
blog_worthy: true
---

# Test Data Pollution Requires Defense-in-Depth, Not Just Cleanup

## What Happened
E2E tests writing to production data stores caused n8n workflows to pick up fake LinkedIn posts. 98% of the queue was test data (51 of 52 items). Individual test cleanup existed but crashed-test survivors accumulated indefinitely. A 3-layer solution was needed: per-spec afterAll cleanup, server-side prefix filtering on GET endpoints, and a Playwright globalTeardown safety net.

## The Insight
Test isolation in shared-data architectures requires the same defense-in-depth mindset as security. No single cleanup layer is reliable enough alone — tests crash, afterAll blocks skip, and stale data accumulates between runs. Server-side filtering (with `?include_test=true` bypass) prevents external consumers from seeing test data even when cleanup fails.

## By The Numbers
- Queue pollution: 51 E2E items removed (98% of queue)
- n8n errors: 14 test entries removed (100% of error log)
- Board tasks: 2 ghost monitoring tasks removed
- Skill timing: 41 e2e entries removed from JSONL
- New endpoints: 3 DELETE routes added for test cleanup

## Pattern
Defense-in-depth test isolation: afterAll cleanup (primary) + server-side prefix filter (secondary) + globalTeardown (safety net)
