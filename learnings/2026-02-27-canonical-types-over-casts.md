---
date: 2026-02-27
source: reflect
issue: "#126"
tags: [architecture, patterns, typescript]
blog_worthy: false
---

# Extend canonical types instead of local cast interfaces

## What Happened
Building a pipeline kanban with drag-and-drop, the server sent fields (`mode_color`, `entry_statuses`) not on the TypeScript `PipelinePhase` type. First approach used a local `RawPhaseData` interface with `as PipelinePhase & RawPhaseData` casts throughout the component. Refactored to add optional fields to the canonical type — eliminated all casts in one change.

## The Insight
When your API already sends extra fields (because it spreads raw JSON), the fix is extending the canonical type, not casting at every usage site. Cast interfaces create N maintenance points; type extension creates 1.

## By The Numbers
- 3 `as ... & RawPhaseData` casts eliminated by adding 3 optional fields to one interface
- 727 lines total in PipelineTab (up from 250) covering 4 features: DnD, actions, mode badges, skill summary

## Pattern
Canonical-type-extension over local-cast-interface
