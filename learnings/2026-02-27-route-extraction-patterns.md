---
date: 2026-02-27
source: reflect
issue: "#106"
tags: [architecture, refactoring, patterns]
blog_worthy: false
---

# Factory Functions Beat Circular Imports for Route Module Extraction

## What Happened
Extracted 4 route domains (DeFi, Calendar, News, Content) from a 3,782-line Express server.js into separate route modules. News routes needed server-level dependencies (API keys, notification functions). Direct imports would create circular dependencies between server.js and routes/news.js.

## The Insight
When extracting route modules that need server-scoped values, a factory function pattern (`createRouter(deps) → { router, init }`) cleanly avoids circular deps and separates module loading from runtime initialization. Timer-based background jobs (polling, caching) must export an `init()` called from `app.listen()` — never fire at import time.

## By The Numbers
- server.js: 3,782 → 1,322 lines (65% reduction)
- 5 new route files totaling 2,462 lines
- 0 endpoint regressions (13 endpoint categories verified via curl)

## Pattern
Factory-init pattern for Express route modules with server-level dependencies
