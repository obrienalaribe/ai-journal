---
date: 2026-02-27
source: reflect
issue: "#105"
tags: [dead-code, patterns, architecture]
blog_worthy: false
---

# Trace Consumers Before Importing During Deduplication Refactors

## What Happened
Replaced 9 local badge components and 7 duplicated constants in Content Engine tabs with imports from shared.tsx. One import (pillarLabel) was unused because it was only consumed by a local component that was being deleted — not directly by JSX. TypeScript's strict mode caught it immediately.

## The Insight
When replacing local code with shared imports, don't just list what the old file declared — trace what the remaining code actually calls. A constant consumed only by a deleted component doesn't need importing.

## By The Numbers
- 720 lines → 401 lines in IdeasTab.tsx (-319 net across 4 files)
- 8 dead types deleted, 2 de-exported, 6 shared.tsx symbols de-exported

## Pattern
Consumer-first import tracing: map call sites in remaining code before writing import statements.
