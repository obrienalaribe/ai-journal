---
date: 2026-03-04
source: refactor
issue: "#236"
tags: [dead-code, dependencies, dx]
blog_worthy: false
---

# Dead code accumulates fastest in rapid multi-phase builds

## What Was Found
After a 4-phase Content Engine Refactor (#236) spanning sidebar extraction, tab merging, LanceDB intelligence backend, and frontend integration, 7 dead code items accumulated: unused exports in extracted components, a dead `embedBatch` helper replaced during iteration, a stale `normalizeScheduleEvents` function from a prior server refactor, and an orphaned `CorrelationResult` type for an unbuilt feature.

## What Was Removed
- 2 unnecessary exports in AnalyticsBadges.tsx (MetricBadge, formatK)
- 2 unnecessary exports in IdeasSidebar.tsx (INTELLIGENCE_STATUSES, PIPELINE_STATUSES)
- 1 dead type in types/index.ts (CorrelationResult)
- 1 dead function in routes/helpers.js (normalizeScheduleEvents)
- 1 dead function + 1 dead constant in routes/intelligence.js (embedBatch, BATCH_SIZE)

## By The Numbers
- knip findings: 7 → 0 (after fixes + 1 false positive confirmed)
- jscpd clones: 14 (all LOW, <10 lines each — not worth extracting)
- API drift: 0 orphan routes, 0 broken consumers
- Files >500 LOC: 12 (architectural, not actionable without feature work)

## Pattern
Multi-phase feature builds that iterate on APIs (embedBatch→embed, StoriesTab→StoriesSection) leave dead code at phase boundaries. Running knip after each phase boundary catches drift before it compounds.
