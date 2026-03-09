---
date: 2026-03-09
source: reflect
issue: "#336"
tags: [hooks, performance, patterns]
blog_worthy: false
---

# Conditional fetching via null URL is safer than dummy endpoint fallback

## What Happened
A custom React fetch hook only accepted `string` URLs, forcing consumers to use dummy fallback endpoints (`?? "/api/stats"`) when a fetch should be skipped. This caused wasted network calls. Adding `string | null` support with an early-return IDLE constant eliminated the hack.

## The Insight
When a fetch hook doesn't support conditional execution, callers will always find workarounds — and those workarounds waste resources silently. Supporting null/disabled URLs at the hook level is cheaper than auditing every consumer for fallback hacks.

## By The Numbers
- 1 unnecessary API call eliminated per intelligence search panel render
- 2 files changed, 17 lines added

## Pattern
Null-URL-as-skip-signal: accept `null` in the hook, return a static IDLE result, reset state in useEffect when URL transitions to null.
