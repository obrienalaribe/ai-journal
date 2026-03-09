---
date: 2026-02-28
source: reflect
issue: "#166"
tags: [debugging, patterns]
blog_worthy: false
---

# Issue specs lie about data paths — verify against actual JSON before coding

## What Happened
Building a composite accumulation score from 6 existing data sources. The issue spec listed field paths like `signals.avgFunding` and `options.putCallRatio` at top level, but actual data nests them as `signals.btc.avgFunding` and `options.putCallRatio.ratio`. Reading the actual JSON file first caught this before any code was written.

## The Insight
Issue specs written from memory or documentation often have stale field paths. A 30-second `python3 -c "import json; ..."` check against real data prevents silent fallback-to-default bugs that are extremely hard to catch in testing.

## By The Numbers
- 6 data source field paths verified, 2 were wrong in the spec
- 0 build errors on first try (thanks to pre-verification)

## Pattern
"Verify before trust" — for any data pipeline work, read actual JSON structure before implementing field access. 30 seconds of verification saves 30 minutes of debugging silent defaults.
