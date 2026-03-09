---
date: 2026-02-28
source: reflect
issue: "#155"
tags: [api, debugging, patterns]
blog_worthy: false
---

# API parameter names don't guarantee the feature works

## What Happened
Built Luma event fetcher with geo-filtering params (lat/lng/radius) that the API silently accepted. Results were globally-ranked regardless of coordinates. Verified by comparing Paris coords request vs. no-coords request — identical results. Scoring provides the actual relevance layer.

## The Insight
Silent acceptance of API parameters is worse than a 400 error. When an API takes your filter params without complaint but doesn't filter, you get data that looks correct but isn't. Always compare filtered vs. unfiltered responses during integration.

## By The Numbers
- 124 Luma events fetched (global, not Paris-specific)
- 19 Brave search results supplementing with location-specific web results

## Pattern
Verify-then-trust: compare filtered vs unfiltered API responses before assuming params work
