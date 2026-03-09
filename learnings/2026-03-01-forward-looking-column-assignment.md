---
date: 2026-03-01
source: reflect
issue: "#185"
tags: [architecture, patterns]
blog_worthy: false
---

# Forward-looking status assignment prevents duplicate Kanban cards

## What Happened
A Kanban board showed cards in two columns because transition statuses (like "approved") were listed as both the exit status of one phase and the entry status of the next. The fix: filter exit statuses that already appear as entry statuses elsewhere, using a forward-looking convention where a status belongs to the phase it enters.

## The Insight
When modeling state machines with phase boundaries, transition statuses must map to exactly one phase. The forward-looking convention (status belongs to the destination phase) is more intuitive because the card's column reflects what's happening next, not what just finished.

## By The Numbers
- 6 duplicate statuses across 7 pipeline phases resolved
- 1 function changed, 4 call sites updated, 0 new components

## Pattern
Forward-looking phase assignment: transition statuses belong to the LATER phase (entry), not the EARLIER phase (exit).
