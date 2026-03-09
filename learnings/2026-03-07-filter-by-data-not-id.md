---
date: 2026-03-07
source: reflect
issue: "#294"
tags: [patterns, architecture]
blog_worthy: false
---

# Filter by Domain Data, Not ID Conventions

## What Happened
ScriptsTab filtered video ideas by `id.startsWith("VID-")` but real ideas use `idea-001` style IDs with YouTube in their `distribution` array. Changed filter to check `distribution.some(d => d.channel === "youtube")` and added cross-tab navigation from Pipeline brief-gen cards to Scripts tab.

## The Insight
ID-prefix-based filtering is brittle and couples display logic to naming conventions. Filter on domain attributes (distribution channels, status fields) instead.

## By The Numbers
- 3 files changed, 38 insertions, 14 deletions
- 0 build errors, 0 iterations needed

## Pattern
Data-attribute filtering over convention-based ID parsing
