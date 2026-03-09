---
date: 2026-02-27
source: reflect
issue: "#133"
tags: [architecture, patterns]
blog_worthy: false
---

# API-first data model: types + routes before any UI

## What Happened
Built Event Radar M1 — TypeScript types, config files, 7 Express API routes, keyword scoring — all without a frontend page. The spec explicitly separated backend (M1) from UI (M2), letting n8n scrapers populate data before the dashboard renders it.

## The Insight
API-first milestones force you to design the contract (types, response shapes, scoring formulas) before visual distractions. The scoring formula bug surface area is zero when you can curl + verify with a calculator, versus debugging it through a React component.

## By The Numbers
- 7 endpoints, 8 types, 3 config files, 1 seed script — 0 iterations needed
- Scoring formula verified: 4 priority matches on test event = score 3 (exact PRD formula match)

## Pattern
API-first milestone decomposition: types + routes + config + seed before any UI code
