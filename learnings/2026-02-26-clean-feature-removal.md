---
date: 2026-02-26
source: reflect
issue: "#80"
tags: [dead-code, patterns]
blog_worthy: false
---

# Clean Feature Removal Requires Test Audit

## What Happened
Removed the Leverage Scenarios calculator section from a DeFi dashboard monitor page. The component (142 lines), its JSX usage, and its dedicated E2E test were all deleted in 3 surgical edits. Build and type checks passed on the first attempt.

## The Insight
Feature removal is only "just delete the code" when you also audit test files. A green build after deletion can mask a now-dead test that would fail if the feature were accidentally re-introduced — false confidence in test coverage.

## By The Numbers
- 180 lines removed (147 component + 33 test)
- 0 build errors, 0 dead imports

## Pattern
Grep-first removal: search all references (src + tests + plans) before any edit, verify zero remaining references after.
