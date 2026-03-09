---
date: 2026-02-26
source: reflect
issue: "#78"
tags: [architecture, patterns, debugging]
blog_worthy: false
---

# Never Trust LLM-Written JSON to Match Your TypeScript Interfaces

## What Happened
A nightly AI agent ran Playwright tests and wrote results to JSON files, but used 5 different schemas across 5 days. The dashboard UI expected `{ passed, failed, skipped }` but got `{ stats.expected, stats.unexpected }` or flat `{ expected, unexpected }` or no counts at all. Every day showed "FAIL" in the activity feed despite tests passing.

## The Insight
When AI agents write structured data, treat them like an untrusted external API: always normalize on read. A 30-line server-side normalizer function is cheaper than debugging schema drift across N agent runs.

## By The Numbers
- 5 different JSON formats across 5 days of agent-written test data
- 1 normalizer function (30 lines) fixed all historical + future data

## Pattern
"Normalize-on-read" — server-side adapter between agent-written data and typed interfaces
