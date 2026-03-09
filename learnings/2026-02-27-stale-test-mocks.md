---
date: 2026-02-27
source: reflect
issue: "#112"
tags: [testing, patterns]
blog_worthy: false
---

# Update test mocks when you rename API fields

## What Happened
A dashboard page crashed because TypeScript interfaces used different field names than the actual API response (`cron_jobs` vs `phase7_cron_efficiency`). After fixing the types and component, /reflect revealed that E2E test mock data still used the old field names - which would silently break at next test run.

## The Insight
When renaming API fields in production types, grep your test directory for the old names. Mock data is typed loosely (plain objects) so TypeScript won't catch stale mocks at compile time - only at test runtime.

## By The Numbers
- 3 files changed: types, component, test mocks
- 1 potential silent test failure prevented by grepping tests/e2e/ for old field names

## Pattern
Schema-change checklist: types -> consumers -> test mocks -> API assertions (all in one atomic change)
