---
date: 2026-02-27
source: reflect
issue: "#115"
tags: [architecture, patterns, dx]
blog_worthy: false
---

# Native Pointer Events Beat DnD Libraries for Single-Axis Drag

## What Happened
Needed drag-to-reschedule for calendar cron events — a single-axis vertical drag within one timeline container. Evaluated dnd-kit, framer Reorder, and HTML5 DnD API. All three were overkill or broken (HTML5 DnD doesn't work on mobile touch). Native Pointer Events API (`pointerdown`/`pointermove`/`pointerup`) delivered zero-dependency, cross-device drag with full snap-to-grid control.

## The Insight
Before reaching for a drag library, check if your drag is single-axis within one container. If yes, native Pointer Events (not the older Mouse/Touch events) give you unified mouse+touch handling with ~40 lines of code and no bundle impact.

## By The Numbers
- 0 new dependencies added (vs 15-40KB for dnd-kit)
- 2 files changed, 249 lines added (backend + frontend)

## Pattern
Native Pointer Events for single-container, single-axis drag; libraries for multi-container or reorder scenarios.
