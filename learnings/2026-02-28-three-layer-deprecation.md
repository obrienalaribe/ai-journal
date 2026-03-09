---
date: 2026-02-28
source: reflect
issue: "#157"
tags: [architecture, patterns, testing]
blog_worthy: false
---

# Deprecating Data Sources Requires Atomic Three-Layer Updates

## What Happened
Event Radar had 5 data sources (Luma, Meetup, Eventbrite, Tavily, Brave) but only 2 were alive. Removing dead sources required synchronized changes across TypeScript union types, backend stats objects, and frontend source metadata. Missing any layer would create ghost UI elements showing "0" for dead APIs or TypeScript allowing invalid source strings.

## The Insight
When a data source type propagates through type definitions, API response shapes, and UI rendering, deprecation must be atomic across all three layers. A partial update creates inconsistency that's hard to detect — TypeScript won't catch runtime shape mismatches, and UI fallbacks silently render "Unknown" for unrecognized sources.

## By The Numbers
- 5 sources narrowed to 2 active sources (Luma + Brave)
- 4 files modified across 3 layers (types, routes, UI, tests)

## Pattern
Three-Layer Atomic Deprecation: when removing a union member, grep all three layers (type definition, backend producer, frontend consumer) and update in one commit.
