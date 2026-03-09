---
date: 2026-03-05
source: reflect
issue: "#270"
tags: [dead-code, architecture, patterns]
blog_worthy: false
---

# Deleting a component orphans its dedicated sub-files

## What Happened
Removed a 530-line IdeaCard component from IdeasTab.tsx (1070 lines down to 395). The TypeScript compiler confirmed zero type errors, but a manual grep revealed AnalyticsBadges.tsx became an orphan file with no remaining consumers.

## The Insight
TypeScript catches unused imports within a file, but not unused files in a project. When deleting a large component, always grep for dedicated sub-files (badges, modals, helpers) that only that component consumed. Use knip or similar for automated detection.

## By The Numbers
- 1070 to 395 lines (-63%)
- 1 orphan file discovered (AnalyticsBadges.tsx)
- 4 E2E tests replaced with 2 updated ones

## Pattern
Cascade deletion audit: component removal requires grep for dedicated sub-files beyond import-level checks.
