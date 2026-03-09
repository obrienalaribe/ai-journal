---
date: 2026-02-28
source: reflect
issue: "#165"
tags: [architecture, patterns]
blog_worthy: false
---

# Multi-path alert delivery beats single-webhook dependency

## What Happened
Aave Health Factor monitoring existed but alerts went to a single webhook URL. If the webhook was misconfigured or down, capital protection alerts were silently lost. Added 3 parallel delivery paths: webhook, gateway notification, and filesystem persistence.

## The Insight
Critical alerts should never depend on a single delivery mechanism. Fan-out to N paths where at least one is guaranteed (filesystem) ensures zero-loss alerting without complex retry logic.

## By The Numbers
- 1 delivery path (webhook only) -> 3 paths (webhook + gateway + file)
- 5 files modified, 113 lines added

## Pattern
Multi-path fire-and-forget alerting with guaranteed persistence fallback
