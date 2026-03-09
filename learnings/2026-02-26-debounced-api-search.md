---
date: 2026-02-26
source: reflect
issue: "#95"
tags: [patterns, hooks, testing]
blog_worthy: false
---

# Debounced API search replaces client-side filtering

## What Happened
Memory Browser search filtered filenames client-side, which was useless for date-stamped files. Moved to server-side content grep with `?q=` param, debounced URL construction via `useDebounce` hook, and snippet extraction (+-2 lines around match). Changed aria-label broke an existing E2E test locator.

## The Insight
When upgrading client-side filters to server-side search, the URL-as-dependency pattern in React hooks is powerful: change the URL string, and `useApi` re-fetches automatically. Debounce the URL, not the fetch call.

## By The Numbers
- 3 files changed, ~40 net lines added
- 6 E2E tests passing (1 new API test added)

## Pattern
URL-driven search: debounce the query string, build URL with params, let the fetch hook react to URL changes.
