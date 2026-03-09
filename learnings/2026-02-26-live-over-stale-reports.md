---
date: 2026-02-26
source: reflect
issue: "#88"
tags: [architecture, patterns]
blog_worthy: false
---

# Live endpoints beat stale report files for operational data

## What Happened
Resource leak counts (worktrees, orphan servers, stale tmux) were read from a daily-generated optimisation report JSON file. By the time the dashboard loaded, the data was hours old. Replaced with a live endpoint that checks the filesystem and process table in real-time.

## The Insight
Operational health data (resource leaks, process counts) should always be live because the whole point is to act on current state. Report files are appropriate for historical analysis (cost trends, PR velocity) where freshness doesn't matter.

## By The Numbers
- 731 → 601 lines in UsageCosts page (Pipeline Health section deleted, Resource Leaks made standalone)
- 1 new endpoint (`GET /api/resources/leaks`) replaces stale JSON reads

## Pattern
Live-check endpoints for operational data; report files for historical/analytical data.
