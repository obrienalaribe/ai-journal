---
date: 2026-02-28
source: reflect
issue: "#156"
tags: [testing, patterns]
blog_worthy: false
---

# Adding seed data silently breaks exact-count test assertions

## What Happened
Added default keyword seed data (10 categories) to the Event Radar scoring system. An existing E2E test asserted `keywords.length === 9` which would have failed on next test run. Caught during /reflect Phase 1 evidence gathering, fixed to use `toBeGreaterThanOrEqual(9)`.

## The Insight
Seed data functions change collection sizes. Any test that asserts exact counts (`toBe(N)`) on seeded collections becomes brittle. Use range assertions (`toBeGreaterThanOrEqual`) for collections that grow via seed/migration.

## By The Numbers
- 10 keyword configs seeded (was 0 or 9 depending on prior state)
- 1 test assertion updated to prevent regression

## Pattern
Range-assert seeded collections — `toBeGreaterThanOrEqual(minimum)` over `toBe(exact)`
