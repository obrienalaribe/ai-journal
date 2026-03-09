---
date: 2026-02-27
source: reflect
issue: "#131"
tags: [patterns, architecture, dx]
blog_worthy: false
---

# Popovers beat modals for contextual actions

## What Happened
Calendar cron events used a full-screen CronDetailModal for view-only job info. Issue #131 needed Run Now + Edit Schedule actions. Instead of adding buttons to the modal, replaced it entirely with an anchored popover that preserves calendar context while adding click-to-act functionality.

## The Insight
When an interaction is contextual (tied to a specific element on screen), popovers maintain spatial context that modals destroy. The user sees the event position while acting on it. Bonus: the implementation is simpler (no focus trap, no overlay, no scroll lock).

## By The Numbers
- CronDetailModal: 140 lines, view-only, 0 actions
- CronPopover: 190 lines, 2 actions (Run Now + Edit Schedule), mobile bottom-sheet
- Net new backend: 2 endpoints, 38 lines

## Pattern
Anchor-to-source popover for contextual CRUD; reserve modals for creation flows and confirmations.
