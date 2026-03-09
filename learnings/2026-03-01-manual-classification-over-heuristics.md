---
date: 2026-03-01
source: reflect
issue: "#187"
tags: [architecture, patterns, dx]
blog_worthy: false
---

# Manual Classification Beats AI Heuristics for Creative Decisions

## What Happened
Content pipeline was auto-assigning platform badges (YouTube, LinkedIn) to raw ideas based on word count and source depth heuristics. OB identified that format choice is a creative decision — the same insight could be a 15-min video OR a LinkedIn post depending on intent. Removed all heuristic classification, replaced with manual channel toggle buttons in the Planning stage.

## The Insight
When the decision requires human judgment (creative, strategic, taste-based), build UI that empowers fast manual input instead of AI recommendations that need overriding. The friction of toggling 4 buttons is lower than the friction of correcting wrong AI suggestions.

## By The Numbers
- 3 files changed, 175 lines added across PipelineTab, IdeaDetailModal, IdeasTab
- 0 build errors on first try — clean implementation from reading files before coding

## Pattern
Manual-toggle-over-heuristic: When outcome depends on human intent, not data patterns, use toggle buttons instead of ML classification.
