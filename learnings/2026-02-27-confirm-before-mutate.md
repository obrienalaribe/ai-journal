---
date: 2026-02-27
source: reflect
issue: "#115"
tags: [patterns, architecture, dx]
blog_worthy: false
---

# Confirmation dialogs should never block — let toasts handle async results

## What Happened
A drag-to-reschedule feature was shipping with immediate PATCH-on-drop (no confirmation). Review requested a confirmation dialog with old/new schedule diff. First instinct was a blocking dialog that stays open until the server confirms — but the revised UX was non-blocking: Accept closes immediately, PATCH fires async, and a toast reports success or rolls back on failure.

## The Insight
For mutation gestures (drag, swipe, reorder), the optimal UX is: confirm intent via dialog → dismiss immediately on accept → optimistic local state → async API call → toast as feedback. This separates "did the user intend this?" from "did the server persist it?" — two distinct concerns that shouldn't block each other.

## By The Numbers
- 3 files changed, 148 insertions, 16 deletions
- 0 TS errors on first build after fixing 2 hook-ordering issues

## Pattern
Optimistic-with-confirmation: dialog confirms intent → optimistic local state → async mutation → toast feedback → data refresh clears optimistic state
