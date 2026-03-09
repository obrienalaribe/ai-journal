---
date: 2026-03-06
source: reflect
issue: "#284"
tags: [patterns, debugging]
blog_worthy: false
---

# Always verify spec endpoint references against actual backend code

## What Happened
Issue #284 specified using `PUT /api/content/ideas/:id` with `{status: "killed"}` to dismiss saved ideas. Reading the actual Express route revealed that endpoint uses a whitelist of allowed fields and `status` is NOT included. The correct endpoint is `PUT /api/content/ideas/:id/status` with separate transition validation logic.

## The Insight
Issue specs written by humans (or AI) reference API shapes from memory. Before implementing any mutation, `grep` the route handler to verify the actual accepted fields and response shape. A wrong endpoint that returns 200 (silently ignoring the field) is worse than one that 404s.

## By The Numbers
- 0 iterations wasted (caught during planning, not debugging)
- 1 file changed, 35 lines added

## Pattern
Verify-before-implement: `grep` the handler, not the spec, for API contract truth.
