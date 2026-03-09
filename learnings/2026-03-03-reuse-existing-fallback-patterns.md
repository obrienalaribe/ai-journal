---
date: 2026-03-03
source: reflect
issue: "#226"
tags: [patterns, architecture]
blog_worthy: false
---

# Reuse existing fallback chains instead of inventing new ones

## What Happened
Needed to add workflow activation checking to an n8n health endpoint. The codebase already had a `/workflows` endpoint with API-first/CLI-fallback logic. Reused that exact pattern in the `/status` endpoint extension instead of writing a new fetch strategy.

## The Insight
When extending an API endpoint, grep for sibling endpoints in the same router first. Existing fallback chains encode hard-won knowledge about what actually works in that environment (which APIs 403, which need CLI, timeout values).

## By The Numbers
- 3 files modified, ~30 LOC backend + ~30 LOC frontend
- 0 build errors, 0 iterations — clean first-try implementation

## Pattern
"Sibling endpoint inheritance" — when adding capabilities to an endpoint, check adjacent endpoints in the same router for reusable patterns before designing from scratch.
