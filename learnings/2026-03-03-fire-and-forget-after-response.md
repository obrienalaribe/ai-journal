---
date: 2026-03-03
source: reflect
issue: "#224"
tags: [architecture, patterns]
blog_worthy: false
---

# Fire-and-forget async work after HTTP response

## What Happened
Content pipeline had a "FULLY AUTO" mode that stopped at approval — requiring manual atom generation and date scheduling. Fixed by adding auto-scheduling (next available slot) in the approve handler and fire-and-forget generation triggered after `res.json()`.

## The Insight
When an HTTP handler needs to trigger expensive async work (AI generation, external API calls), send the response first, then fire-and-forget. The key safety requirement: ensure the async operation's failure leaves the system in a recoverable state, not an inconsistent one.

## By The Numbers
- 86 lines added to content.js (2 functions + handler modification)
- 1 E2E test assertion updated (null → truthy for auto-filled publish_at_utc)

## Pattern
Response-first fire-and-forget: `res.json(result); asyncWork().catch(log)`
