---
date: 2026-02-27
source: reflect
issue: "#117"
tags: [architecture, patterns]
blog_worthy: false
---

# Causation Events Beat Price Predictions in Market Dashboards

## What Happened
Built a market intelligence dashboard showing Polymarket prediction markets alongside derivatives data. Initial version mixed price prediction events ("Will BTC hit $100K?") with causation events ("Fed decision in March?"). Stakeholder review flagged this: the price is already on the chart — we need the WHY, not another copy of the WHAT.

## The Insight
When building financial dashboards, filter for causation events (catalysts that CAUSE price movement) over actualization events (predictions of the movement itself). Users have 10 price feeds already — alpha comes from understanding catalysts.

## By The Numbers
- 177 lines removed, net -165 lines across 4 files
- Polymarket events reduced from mixed to catalyst-only (Fed chair, rate decisions, recession risk)

## Pattern
Causation > Actualization: filter data feeds for upstream causes, not downstream predictions of the same metric shown elsewhere.
