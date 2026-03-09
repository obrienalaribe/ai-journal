---
date: 2026-02-26
source: reflect
issue: "#85"
tags: [architecture, debugging, patterns]
blog_worthy: false
---

# Visual Overlap Detection: When Pixels Disagree With Time

## What Happened
Calendar events with 1-minute duration rendered as 28px blocks (minimum height), but the overlap detection algorithm used time-based end times. Two cron jobs 2 minutes apart didn't overlap in time, but their 28px blocks overlapped visually, stacking on top of each other instead of rendering side-by-side.

## The Insight
When UI elements have minimum rendered dimensions that exceed their logical data range, any layout algorithm that checks for collisions must use the visual bounds, not the data bounds. This applies to any timeline, Gantt chart, or positioning system with min-height/min-width constraints.

## By The Numbers
- 2 cron events at 06:00/06:02 now correctly render in 2-column grid
- 28px min-height = 0.467 hours of visual space for a 1-minute event (28x larger than actual)

## Pattern
Visual-bounds collision detection: `visualEnd = max(dataEnd, dataStart + minVisualSize / scale)`
