---
date: 2026-02-28
source: reflect
issue: "#175"
tags: [testing, patterns, debugging]
blog_worthy: false
---

# Nulling tokens on success breaks idempotent replay

## What Happened
A queue system set `claim_token: null` after successful posting, but the idempotent replay check compared `item.claim_token === request_token`. Since null !== UUID, replays returned 403 instead of 200. The fix: check terminal state (posted/failed) regardless of token match.

## The Insight
Any "cleanup" that nullifies auth tokens on success creates a silent replay bug. If your system promises idempotent callbacks, the replay path must check outcome state, not auth state.

## By The Numbers
- 2 bugs found by code review before any test ran
- 3 test iterations to handle Playwright serial mode + stale data cleanup
- 9 E2E tests covering full queue lifecycle (approve through posted)

## Pattern
Terminal-state replay: check `if (status in terminalStates) return replay` before checking auth tokens.
