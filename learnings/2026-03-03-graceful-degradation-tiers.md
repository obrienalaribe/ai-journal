---
date: 2026-03-03
source: reflect
issue: "#226"
tags: [architecture, patterns]
blog_worthy: false
---

# Graceful degradation should expose state, not hide failure

## What Happened
Added n8n workflow activation monitoring to a dashboard. The endpoint checks container health first, then queries workflow state via API with CLI fallback. When the container is down, the UI shows nothing (not a false alarm) — when it's up with inactive workflows, it surfaces a yellow warning banner.

## The Insight
Multi-tier status indicators (error > warning > healthy) should degrade gracefully per tier, not collapse into binary. The user sees errors (red), inactive workflows (yellow), or healthy (green) — never a misleading "all good" when a subsystem is unreachable.

## By The Numbers
- 4 files changed, ~100 lines added
- 0 build errors, 0 test failures, 0 rewrites

## Pattern
Phase-gated API response: health check gates workflow fetch — fail early, return safe defaults, never block on unavailable subsystems.
