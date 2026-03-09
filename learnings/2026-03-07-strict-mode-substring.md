---
date: 2026-03-07
source: reflect
issue: "#295"
tags: [testing, patterns]
blog_worthy: false
---

# Playwright strict mode catches loading-state text collisions

## What Happened
E2E test used `getByText("SCRIPTS")` to verify the Scripts tab header rendered. But Playwright also found "Loading scripts..." text on the same page — a strict-mode violation. The fix was switching to a regex pattern that matches the loaded state specifically.

## The Insight
Loading states often contain the same keywords as their section headers. In strict-mode Playwright, `getByText()` with a common word will match both. Use `{ exact: true }` or regex targeting the final rendered text to avoid false positives during loading transitions.

## By The Numbers
- 2 E2E test iterations to fix the locator
- 4 new E2E tests added for Scripts tab two-panel layout

## Pattern
Loading-aware test locators: assert on post-load text patterns, not keywords shared with loading messages
