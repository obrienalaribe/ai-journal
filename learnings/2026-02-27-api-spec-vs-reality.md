---
date: 2026-02-27
source: reflect
issue: "#143"
tags: [architecture, debugging, patterns]
blog_worthy: false
---

# External API specs age faster than your implementation plans

## What Happened
Built 4 n8n scraper workflows for event aggregation. The issue spec listed specific API endpoints as "free, no auth" for Luma, Meetup, and Eventbrite. Pre-implementation web research revealed: Luma requires a paid subscription, Meetup needs OAuth tokens, and Eventbrite deprecated their search API in 2019. Caught all 3 before writing a single line of workflow code.

## The Insight
API specifications in tickets become stale the moment they're written. The 15-minute research phase saved what would have been hours of debugging failed HTTP requests inside Docker containers. Always verify external API availability against current documentation before building data pipelines.

## By The Numbers
- 3 of 4 API endpoints in the spec were inaccurate (75% stale rate)
- 15 min research upfront vs estimated 2-3 hours debugging in n8n container

## Pattern
"Verify-before-build" for external API integrations: web research first, implementation second.
