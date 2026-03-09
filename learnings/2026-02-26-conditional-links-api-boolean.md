---
date: 2026-02-26
source: reflect
issue: "#78"
tags: [architecture, patterns, dx]
blog_worthy: false
---

# Don't let UIs link to resources that might not exist

## What Happened
A dashboard showed "VIEW REPORT" links on every test result card, but only today's report was archived. All historical links returned 404. The fix was adding a `hasReport` boolean to the API response, computed by checking filesystem presence server-side, then conditionally rendering the link.

## The Insight
When UI elements link to server-side resources (files, reports, archives), the API should include a presence boolean. Let the server — which owns the truth — tell the client what exists. Never let the client assume availability.

## By The Numbers
- 5 test cards rendered, only 1 had a valid report → 80% of links were broken
- Fix: +5 lines server, +1 line UI, +1 line type = 7 total lines

## Pattern
Server-side resource presence booleans over client-side assumption
