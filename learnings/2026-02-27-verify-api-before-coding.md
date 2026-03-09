---
date: 2026-02-27
source: reflect
issue: "#116"
tags: [api, debugging, patterns]
blog_worthy: false
---

# Always curl external APIs before writing integration code

## What Happened
Integrating Polymarket prediction markets into a DeFi dashboard. The spec described tags as simple strings but the actual API returns objects with a `label` field. A pre-coding curl test caught this before writing filtering code that would have silently matched nothing.

## The Insight
External API documentation (or issue specs based on docs) can differ from actual response shapes. A 30-second curl test before coding prevents hours of debugging silent filter failures.

## By The Numbers
- 266 lines of new code across 3 files
- 1 tsc error caught and fixed (unused variable)
- 8 relevant events filtered from 35 total using verified tag/keyword logic

## Pattern
Curl-first integration: verify response shape with actual API call before writing any transformation code.
