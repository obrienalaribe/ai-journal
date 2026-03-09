---
date: 2026-02-26
source: reflect
issue: "#81"
tags: [architecture, debugging]
blog_worthy: false
---

# Volume profiles need recency windows, not full history

## What Happened
BTC OHLCV chart showed support at $22-23K for a $68K asset. The volume-profile algorithm used 5.5 years of daily candles, so 2022 bear-market high-volume zones dominated the profile, producing meaningless support levels 65% below current price. Fixed by filtering to last 180 days.

## The Insight
When computing derived signals from time-series data (support/resistance, moving averages, volatility), always question the lookback window. Full history biases toward extinct regimes. A fixed recency window (180d) eliminates noise while capturing current market structure.

## By The Numbers
- Support lines: 2 ($22K ancient) -> 1 ($66K relevant)
- Y-axis range: $10K-$130K -> natural compression around recent prices
- Code change: 12 insertions, 9 deletions across 2 files

## Pattern
Recency-windowed aggregation over full-history aggregation for derived signals
