---
date: 2026-02-27
source: refactor
issue: "#105"
tags: [dead-code, dependencies]
blog_worthy: false
---

# Planned API surfaces become dead types when no consumer materializes

## What Was Found
knip flagged 2 exported types (PipelinePhaseId, QualityRubric) with zero consumers. These were "planned API surface" types created during Content Engine implementation in Feb 2026 but never imported by any component.

## What Was Removed
- 2 dead exported types from src/types/index.ts (PipelinePhaseId, QualityRubric)
- Combined with the implementation phase: 8 total Content types removed/de-exported

## By The Numbers
- knip findings: 2 → 0
- types/index.ts: 1193 → 1175 lines (-18)
- Total session: -385 lines across 5 files

## Pattern
Run knip after every Content Engine PR — speculative type exports accumulate faster than consumers
