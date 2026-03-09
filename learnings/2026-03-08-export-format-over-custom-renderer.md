---
date: 2026-03-08
source: reflect
issue: "#307"
tags: [architecture, 10pct, patterns]
blog_worthy: true
---

# When "Export as X" exists, improve the export — don't build a parallel renderer

## What Happened
Built a 423-line custom HTML flowchart component (NarrativeMindMap.tsx) to visualize script narrative flow in-app. The dashboard already had an Excalidraw JSON export button. The user's actual goal was better Excalidraw output (one bullet per node, presentation-ready). The HTML renderer solved a problem the user didn't ask for while the Excalidraw JSON stayed at lane-level granularity.

## The Insight
When a feature already exports to a specialized format (Excalidraw, SVG, PDF), improving the export generator is almost always the 10% solution. Building a custom in-app renderer creates two maintenance paths, duplicates rendering logic, and produces an inferior version of what the specialized tool already does. Ask: "does a custom renderer add value that the export tool cannot provide?" If the answer is "convenience," that's not enough to justify 400+ lines.

## By The Numbers
- 423 lines of NarrativeMindMap.tsx vs ~80 estimated lines to upgrade Excalidraw generator
- Parser tested against 1 of 18 briefs during development; 6 of 18 hit degraded prose fallback
- 5:1 code ratio (built vs needed) — violated the project's 10% Rule

## Pattern
Export-first over render-first: when users need visual output for external tools, optimize the export format. In-app preview is optional and should reuse the export, not duplicate it.
