---
date: 2026-02-26
source: reflect
issue: "#88"
tags: [patterns, architecture]
blog_worthy: false
---

# Conditionally rendered sections beat loading states for optional data

## What Happened
Added optimisation report sections to an existing dashboard page. The report data comes from a daily cron (06:00) via a generic API endpoint. Instead of showing loading/error states for the new sections, used conditional rendering: sections only appear when data exists.

## The Insight
When a page has a primary data source (required) and secondary enrichment sources (optional), render the secondary sections conditionally rather than blocking the page or showing error states. Users see the core page immediately; enrichment sections appear when available.

## By The Numbers
- 380 → 731 lines in page component (2 new sections + 3 helper components)
- 80 new type definitions across 10 interfaces

## Pattern
Conditional section rendering for optional enrichment data — `{data && <Section />}` over `{loading ? <Loading /> : <Section />}`
