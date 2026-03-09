---
date: 2026-02-28
source: reflect
issue: "#167"
tags: [architecture, debugging, patterns]
blog_worthy: false
---

# Always Fetch the Actual API Before Trusting a Spec

## What Happened
Issue spec recommended fetching Aave governance proposals via Discourse `/latest.json`. WebFetch revealed that endpoint returns 30 generic topics with no excerpts and no keyword filtering. The `/search.json?q=keyword` endpoint was dramatically better: pre-filtered results, blurb excerpts, and up to 50 results per query. Switching before writing any code saved at least one full iteration cycle.

## The Insight
Specs describe intent, not implementation. A 30-second API probe can invalidate an entire approach. Always `curl` (or WebFetch) the actual endpoint with real params before committing to an architecture.

## By The Numbers
- 316 lines total implementation (script + route + types + UI)
- 0 iterations needed on API integration (correct on first attempt)

## Pattern
Probe-before-build: WebFetch/curl the external API response shape before writing any integration code.
