---
date: 2026-03-01
source: reflect
issue: "#205"
tags: [architecture, patterns, debugging]
blog_worthy: false
---

# Normalize on read beats validate on write for schemaless JSON stores

## What Happened
Content ideas stored as JSON files were missing fields (source_ids, keywords, quality_gates) causing frontend crashes. Multiple writers (API, cron, AI agent) bypassed schema enforcement. Added a normalizeIdea() function applied at all 6 API response sites.

## The Insight
When your data store has no schema enforcement (JSON files, NoSQL), normalize on read is more robust than validate on write. Write-side validation only protects known entry points. Read-side normalization defends against all data corruption regardless of source.

## By The Numbers
- 6 API response sites needed normalization (initially found 2, grep found 4 more)
- 18 existing ideas with missing fields, all now served with complete schemas

## Pattern
Normalize-on-read: pure function that fills defaults using nullish coalescing, applied via .map() on every GET response.
