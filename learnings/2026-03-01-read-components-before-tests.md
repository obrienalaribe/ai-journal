---
date: 2026-03-01
source: reflect
issue: "#183"
tags: [testing, debugging, patterns]
blog_worthy: false
---

# Always Read UI Components Before Fixing Their Tests

## What Happened
10 E2E tests failed after recent feature work. Fixing content.spec.ts took 3 iterations because I assumed the UI still rendered "TRIAGE" text — the component had been refactored to show "Intake Empty" for empty state and "Filter ALL" button for populated state. Reading the component source first would have been one-and-done.

## The Insight
When E2E tests break, the root cause is often a component behavior change, not a test logic error. Read the component source before editing the test — it saves iteration cycles.

## By The Numbers
- 11 failures → 0 failures across 3 spec files
- 3 iterations on content.spec.ts IdeasTab locators vs 1 iteration needed

## Pattern
Component-first test debugging: read source → identify actual DOM → fix locator
