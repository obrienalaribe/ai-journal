---
date: 2026-02-27
source: reflect
issue: "#129"
tags: [architecture, patterns, debugging]
blog_worthy: false
---

# writeFileSync Needs Its Parents

## What Happened
Building a 3-tier monitoring system for n8n workflow failures (Telegram → Dashboard → NEXUS escalation). The POST endpoint that receives errors and escalates to board.json crashed with ENOENT because the tasks directory didn't exist in the test environment. readJSON gracefully returns null for missing files, but writeFileSync has no such forgiveness.

## The Insight
In filesystem-backed architectures, read paths are forgiving (return null/default) but write paths are strict (throw on missing parent). Every write to a shared directory constant needs an ensureDir guard, not just read-then-write patterns.

## By The Numbers
- 7 files modified, ~130 lines of new code
- 3 new API endpoints added
- 1 runtime crash caught and fixed during manual verification

## Pattern
ensureDir-before-write: Always call ensureDir(parentDir) before writeFileSync to a path under a shared directory constant, even if readJSON works fine on the same path.
