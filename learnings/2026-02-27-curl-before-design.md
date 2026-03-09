---
date: 2026-02-27
source: reflect
issue: "#117"
tags: [architecture, debugging, patterns]
blog_worthy: false
---

# Test APIs From Deployment Before Designing Pipelines

## What Happened
Needed to replace geo-blocked Binance derivatives data with alternatives. Tested 5 exchange APIs from the actual server before writing any code. Bybit returned 403 (CloudFront block) -- same geo-restriction as Binance. OKX and Deribit worked perfectly. Saved hours of implementation time by discovering this in 30 seconds of curl testing.

## The Insight
API documentation tells you what endpoints exist. Only a curl from your actual deployment server tells you which ones work. Geo-blocking and IP restrictions are invisible in docs but fatal during implementation.

## By The Numbers
- 5 exchange APIs tested, 2 blocked (Binance 451, Bybit 403), 3 working (OKX, Deribit, CoinGecko)
- 12 individual endpoints verified with actual HTTP responses
- 0 lines of code written before validation complete

## Pattern
"Smoke-test infrastructure before architecture" -- always validate external dependencies from the deployment environment before designing systems around them.
