---
date: 2026-03-04
source: reflect
issue: "#236"
tags: [architecture, debugging, patterns]
blog_worthy: false
---

# Schema Evolution in Embedded Databases Is Not Free

## What Happened
Built a 4-layer intelligence engine using LanceDB with a rich schema (14 columns, 11 indices). After adding new columns (vector_rich, viral_score, severity) to the table creation code, search queries broke silently — LanceDB's `.select()` rejects columns not physically present in existing tables. The table was created by an earlier version with fewer columns.

## The Insight
Embedded columnar databases (LanceDB, DuckDB, SQLite) don't auto-migrate schemas like ORMs do. Adding columns to creation code doesn't affect existing tables. A production schema evolution strategy (addColumns migration or drop-and-recreate) must be designed at Day 1, not discovered at deployment time.

## By The Numbers
- 14 schema columns defined, but only 10 existed in the running table
- 3 search queries returned 0 results silently before the mismatch was caught
- 39/39 E2E tests passing after fix (was 0 search results before)

## Pattern
Defensive column projection — only select columns confirmed in actual table, derive missing fields from existing data (e.g. tags from tags_text).
