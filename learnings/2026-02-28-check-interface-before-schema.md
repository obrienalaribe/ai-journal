---
date: 2026-02-28
source: reflect
issue: "#163"
tags: [architecture, patterns]
blog_worthy: false
---

# Check the TypeScript interface before designing the data schema

## What Happened
Built a backend data pipeline (JSONL timing entries) for a frontend Gantt chart. Included ISO date strings but missed numeric timestamp fields (`startMs`/`endMs`) that the chart required for pixel positioning. Caught during self-reflection, not during initial testing.

## The Insight
When building backend data for a frontend visualization, the TypeScript interface is the contract. Reading it before writing any data-producing code prevents schema mismatches that only surface when the UI renders blank.

## By The Numbers
- 36 seed entries fixed (added startMs/endMs to all)
- 1 Living Rule added to prevent recurrence

## Pattern
Interface-first data design: read the consumer's type definition before writing the producer.
