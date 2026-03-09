---
date: 2026-02-27
source: reflect
issue: "#117"
tags: [architecture, patterns, debugging]
blog_worthy: false
---

# Research-first approach eliminates API integration trial-and-error

## What Happened
Building a multi-source market intelligence feature (Polymarket + OKX + Deribit), we spawned a dedicated research agent to verify API availability, geo-blocking, rate limits, and response shapes before writing any code. The research agent discovered Bybit is geo-blocked (403) from this server and documented exact OKX param gotchas (ccy vs instId, period=5m vs 1H) that would have cost 2-3 debug cycles.

## The Insight
When integrating 3+ external APIs, spending 15 minutes on a structured research phase (curl every endpoint, document response shapes, verify auth requirements) saves 2-3x that in trial-and-error debugging. The research document becomes the spec the implementer codes against.

## By The Numbers
- 3 APIs integrated with 0 runtime errors on first try
- 8 API calls per refresh cycle, reduced to 1-3 via 3-tier caching
- 691 lines added across 3 files, 0 type errors

## Pattern
Research-agent-to-implementer handoff via structured markdown spec
