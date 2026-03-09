---
date: 2026-03-01
source: reflect
issue: "#189"
tags: [architecture, debugging, patterns]
blog_worthy: false
---

# Express Parameterized Routes Silently Swallow Literal Siblings

## What Happened
Added a `/schedule/oplog` GET endpoint for the idea detail modal's activity log. Initially placed it after `/schedule/:week` in the file. Curl returned week schedule JSON instead of oplog array — the `:week` parameter captured "oplog" as a week string.

## The Insight
Express route matching is order-dependent and parameterized routes are greedy. This is a well-known pattern but easy to forget when a file has 2000 lines and the parameterized route is 200 lines away from your new one. Always grep for `/:param` siblings before registering literal routes.

## By The Numbers
- 1 route shadowing bug caught during manual verification (would have shipped broken)
- 0 tsc/build warnings for route ordering (invisible at compile time)

## Pattern
"Literal-before-parameterized" — grep for parameterized siblings before any Express route addition
