---
date: 2026-02-25
source: reflect
issue: "#73"
tags: [architecture, patterns, debugging]
blog_worthy: false
---

# Pipeline phases and content statuses are many-to-many

## What Happened
When aligning a content engine kanban board to a PRD spec, the original code assumed a 1:1 mapping between pipeline phases (7 columns) and content statuses. The PRD defines 13+ granular statuses distributed across 7 phases. Changing column names broke filtering because `status === phaseId` no longer matched. The fix was adding a `statuses[]` array to each phase definition and filtering with `phaseStatuses.includes(status)`.

## The Insight
When a domain model has hierarchical groupings (phases containing statuses), encode the mapping in data rather than code. A `statuses[]` field in the phase configuration makes the mapping explicit, queryable, and changeable without code deployments.

## By The Numbers
- 7 pipeline phases → 15 content statuses (13 active + parked + killed)
- 4 type definitions updated (ContentPillar, ContentAngle, Platform, ContentStatus)
- 3 new E2E tests covering pipeline API shape and PRD-aligned defaults

## Pattern
Data-driven grouping: hierarchical entities use explicit membership arrays rather than naming conventions or positional inference.
