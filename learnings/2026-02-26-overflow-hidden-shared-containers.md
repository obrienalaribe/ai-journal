---
date: 2026-02-26
source: reflect
issue: "#82"
tags: [architecture, patterns]
blog_worthy: false
---

# Shared container primitives should never clip overflow by default

## What Happened
A shared Card component had `overflow: hidden` as a default style, which silently clipped all position:absolute tooltips rendered inside it. z-index couldn't escape the parent's overflow boundary. Removing the default and letting consumers opt-in via the style prop fixed all tooltip visibility issues with zero regressions.

## The Insight
Shared layout primitives should be permissive by default. Restrictive CSS properties like `overflow: hidden` should be opt-in, not opt-out, because they create invisible constraints that downstream components can't override without restructuring the DOM.

## By The Numbers
- 1 line removed from Card.tsx, 0 regressions across 54 passing E2E tests
- 3 bugs fixed in 2 files, 7 insertions / 8 deletions total

## Pattern
Permissive-default containers with opt-in restrictions
