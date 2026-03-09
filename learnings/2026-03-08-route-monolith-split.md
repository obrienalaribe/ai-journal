---
date: 2026-03-08
source: reflect
issue: "#318"
tags: [architecture, refactor, patterns]
blog_worthy: false
---

# When splitting a monolith, shared helpers beat duplicated constants

## What Happened
A 3976-line Express route file needed splitting into 3 focused modules (photos, schedule, content). The initial plan said "no shared module — each file defines its own path constants." During implementation, this was correctly abandoned: 17 path constants and 30+ helper functions were used across all 3 modules. A domain-specific `content-helpers.js` (723 lines) was created as a shared module.

## The Insight
When splitting a monolith, always audit the dependency graph before choosing an architecture. The "duplicate everything" approach looks simpler but creates drift risk — one path constant changing in one file but not another is a production bug waiting to happen. A shared helpers module adds one file but prevents N constant-drift bugs.

## By The Numbers
- 3976 → 1099 + 217 + 1511 + 723 lines (4 files, same total)
- 81 API routes preserved (0 diff in path audit)
- 7 over-exported symbols identified for /refactor cleanup

## Pattern
Domain-specific shared helpers module (`{domain}-helpers.js`) alongside project-wide `helpers.js`
