---
date: 2026-02-28
source: reflect
issue: "#176"
tags: [architecture, patterns, debugging]
blog_worthy: false
---

# Idempotency Keys Must Outlive Claim Tokens

## What Happened
A content delivery queue nulled the claim_token after successful posting for security. But the idempotent replay check compared claim_token from the replay request against the stored (now null) value, causing 403 instead of 200 on legitimate replays.

## The Insight
In any claim/lease queue system, the idempotency key and the claim token serve different lifetimes. The claim proves ownership during processing; the idempotency key proves identity across retries. Never use one for the other's job.

## By The Numbers
- 6 bugs fixed in one session (idempotency, timezone, backoff, auth, rate limit, per-channel)
- 16 E2E tests covering full queue lifecycle + hardening

## Pattern
Separate identity keys (idempotency_key, stable across retries) from ownership keys (claim_token, ephemeral during processing).
