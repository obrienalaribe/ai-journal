---
date: 2026-02-28
source: reflect
issue: "#168"
tags: [architecture, patterns, debugging]
blog_worthy: false
---

# Derive formulas algebraically before coding — especially for financial math

## What Happened
Needed to calculate Aave V3 liquidation price from on-chain data. Issue spec provided a formula involving collateral amounts and debt. Instead of implementing the raw formula (which requires fetching BTC amount separately), derived it algebraically: liqPrice = btcPrice / healthFactor. This eliminated one variable and reduced to a single external API call.

## The Insight
Financial formulas often simplify dramatically when you substitute known intermediate values (like Health Factor, which Aave already computes). The simplified form is also easier to unit test and reason about correctness.

## By The Numbers
- 3 API calls (debt + collateral + btcPrice) reduced to 1 (btcPrice only)
- 1 formula line vs 4 intermediate computation lines

## Pattern
Algebraic simplification before implementation — substitute known composites (HF, ratios) to reduce the number of runtime dependencies.
