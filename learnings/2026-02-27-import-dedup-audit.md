---
date: 2026-02-27
source: reflect
issue: "#105"
tags: [dead-code, patterns, architecture]
blog_worthy: false
---

# Shared module imports need consumer-level tracing, not name matching

## What Happened
Content Engine had 9 local badge components and 7 duplicate constants in IdeasTab that drifted from the canonical shared.tsx versions. Replacing locals with shared imports initially pulled in 4 symbols (ANGLE_COLORS, pillarLabel, statusLabel, angleLabel) that were only used internally by shared components, not by the consumer.

## The Insight
When deduplicating toward a shared module, trace each import to its actual call site in the consumer. Shared components consume their own internal helpers — importing those helpers into the consumer is unnecessary and creates false coupling.

## By The Numbers
- 720 lines reduced to 498 (IdeasTab: -222 lines, SourcesTab: -50 lines)
- 8 dead type exports removed from types/index.ts
- 6 shared.tsx exports de-exported (internal-only)

## Pattern
Consumer-level import tracing over mechanical name deduplication
