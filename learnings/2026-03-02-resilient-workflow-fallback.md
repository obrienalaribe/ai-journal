---
date: 2026-03-02
source: reflect
issue: "#195"
tags: [architecture, testing, patterns]
blog_worthy: false
---

# Resilient workflow fallback: defensive file reads in automation pipelines

## What Happened
n8n LinkedIn posting workflow assumed file reads always succeed after server-side validation. Added `continueOnFail` + conditional routing so file read failures fall back to text-only posting instead of failing the entire queue item. Added 4 E2E tests covering the photo_valid claim field behavior.

## The Insight
When automation pipelines validate state at claim time but act on it later, add defensive checks at the action step too. The time gap between validation and execution is a race condition window.

## By The Numbers
- 30 -> 34 content-delivery E2E tests
- 2 n8n workflow nodes added (Photo Read OK? IF + continueOnFail flag)

## Pattern
Validate-then-act pipelines need defense-in-depth: validate at claim + defensive check at execution + graceful fallback path.
