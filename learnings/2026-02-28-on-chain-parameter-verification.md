---
date: 2026-02-28
source: reflect
tags: [architecture, debugging, patterns]
blog_worthy: false
---

# Verify DeFi Parameters On-Chain Before Building Strategies

## What Happened
Research session needed cbBTC risk parameters (LTV, liquidation threshold) for Aave V3. Web searches returned vague results without specific numbers. Querying the Aave V3 PoolDataProvider contract directly via RPC returned exact on-chain parameters in seconds: LTV=73%, LiqThreshold=78%, LiqBonus=7.5%.

## The Insight
DeFi protocol parameters live on-chain and can be queried via standard RPC calls with known function selectors. This is faster and more reliable than searching governance forums or documentation. The pattern works for any EVM protocol with public view functions.

## By The Numbers
- Web search: 4 queries, 0 specific parameter values returned
- On-chain query: 1 RPC call, all 10 parameters returned in <1 second

## Pattern
On-chain-first parameter lookup: identify the protocol's data provider contract, call the appropriate view function (e.g., `getReserveConfigurationData`), decode the return values. Skip documentation/governance searches for live parameter values.
