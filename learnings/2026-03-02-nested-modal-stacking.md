---
date: 2026-03-02
source: reflect
issue: "#210"
tags: [architecture, patterns, ui]
blog_worthy: false
---

# Nested modals need explicit z-index stacking and Escape handler ordering

## What Happened
Redesigned a Generated Content section from flat 20-item list to grouped-by-channel compact rows with platform-specific preview modals. The preview modal (LinkedIn/YouTube/Blog simulation) opens on top of the detail modal, creating a nested modal pattern. Required explicit z-index separation (210 vs 200) and awareness that both register document-level Escape handlers.

## The Insight
When stacking modals, Escape key handling works correctly via React's render lifecycle: the inner modal unmounts first (clearing its listener), then the outer modal's listener runs on the next keypress. No special coordination needed if inner modal is rendered inside the outer modal's tree.

## By The Numbers
- IdeaDetailModal: 828 -> 910 lines (+82 net, -51 removed flat list, +133 grouped display + preview wiring)
- PlatformPreviewModal: 407 lines (new file)
- 0 new backend endpoints needed (reused PUT /atoms/:id)

## Pattern
Nested modal z-index stacking with React unmount ordering
