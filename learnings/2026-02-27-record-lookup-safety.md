---
date: 2026-02-27
source: reflect
issue: "#145"
tags: [typescript, patterns]
blog_worthy: false
---

# TypeScript Record Lookups Need Safe Accessors

## What Happened
Built an event page with 10 category lookup tables using `Record<string, T>`. Strict mode flagged 25 errors because `Record[key]` returns `T | undefined`. Fixed by extracting safe accessor functions with `?? DEFAULT` fallback.

## The Insight
When using `Record<string, T>` as a lookup table in strict TypeScript, always wrap index access in a named function that returns a guaranteed `T`. This prevents undefined propagation through deeply nested JSX property chains.

## By The Numbers
- 25 type errors from a single root cause (Record index access)
- Fixed with 2 helper functions (4 lines) + find-and-replace

## Pattern
Safe Record Accessor — `function getMeta(id: string) { return LOOKUP[id] ?? DEFAULT; }`
