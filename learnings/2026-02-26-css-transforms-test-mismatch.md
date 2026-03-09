---
date: 2026-02-26
source: reflect
issue: "#89"
tags: [testing, patterns, debugging]
blog_worthy: false
---

# CSS text-transform tricks Playwright's text matchers

## What Happened
Added type badges (Daily/Reflection/Reference) to a timeline view with CSS `text-transform: uppercase` for visual styling. E2E tests used uppercase regex patterns (`/^DAILY$/`) which failed because Playwright matches DOM text content, not CSS-rendered visual text.

## The Insight
Playwright's `getByText()` operates on the accessibility tree and DOM text nodes, not on the CSS-rendered output. Any CSS transformation (uppercase, capitalize, lowercase) is invisible to text matchers. Always match against the actual text content in the source code.

## By The Numbers
- 5 new E2E tests added for Memory Browser (was 0 coverage)
- 1 test rewrite to fix text-transform mismatch

## Pattern
Match DOM text, not rendered text — CSS transforms are invisible to test frameworks.
