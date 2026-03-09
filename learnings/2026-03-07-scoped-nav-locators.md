---
date: 2026-03-07
source: reflect
issue: "#295"
tags: [testing, patterns, debugging]
blog_worthy: false
---

# Scope Playwright nav locators to avoid content-page text collisions

## What Happened
E2E tests for a new Scripts tab redesign failed 5 times due to strict mode violations. `getByRole("button", { name: /content/i })` matched 4 elements because the Activity Feed page displays commit messages containing "content" inside clickable elements. Resolved by scoping to `page.locator("nav").getByText("LABEL", { exact: true })`.

## The Insight
On content-heavy dashboards, generic button text (like "CONTENT") will collide with dynamically rendered page content. Always scope navigation locators to the sidebar/nav container rather than searching the full page.

## By The Numbers
- 5 test iterations to resolve locator → 1 with scoped approach
- 30 existing tests still pass after the fix

## Pattern
Container-scoped locators for navigation elements on data-dense pages
