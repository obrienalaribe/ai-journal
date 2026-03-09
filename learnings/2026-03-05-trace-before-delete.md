---
date: 2026-03-05
source: reflect
issue: "#241"
tags: [dead-code, patterns]
blog_worthy: false
---

# Trace Three Layers Before Removing Init Functions

## What Happened
Removed a startup seeder function (seedContentFiles) that created config files and seeded demo data. Before deleting, traced three layers: constants it defined (3 orphaned), imports it consumed (1 still used elsewhere), and files it created (2 already existed on disk). Clean removal with zero build errors.

## The Insight
Init/seed functions are dependency hubs — they reference constants, imports, and filesystem paths that may or may not have other consumers. A 3-layer trace (constants → imports → artifacts) prevents orphaned dead code from surviving the removal.

## By The Numbers
- 264 lines deleted across 3 files
- 5 orphaned constants/functions removed (3 constants + 2 functions)

## Pattern
Three-layer trace: constants defined → imports consumed → artifacts created
