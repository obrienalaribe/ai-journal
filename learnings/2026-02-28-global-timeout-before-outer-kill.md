---
date: 2026-02-28
source: reflect
issue: "#159"
tags: [ci, testing, performance]
blog_worthy: false
---

# Set inner timeouts before outer kill signals

## What Happened
128 Playwright tests ran during nightly maintenance with a 5-minute outer process kill but no Playwright-level global timeout. The outer kill fired mid-suite, producing no JSON results and triggering a GitHub issue about test infrastructure being broken.

## The Insight
When CI wraps test runners in a process-level timeout (`timeout 300`), always set an inner `globalTimeout` lower than the outer boundary. The inner timeout lets the runner exit cleanly with proper results, while the outer timeout remains a safety net.

## By The Numbers
- 128 tests across 12 spec files — worst-case sequential: 64 min
- Fix: 3 config lines (globalTimeout + fullyParallel + maxFailures)

## Pattern
Inner-timeout-before-outer-kill: any process with a hard kill boundary should self-terminate cleanly at 80% of that boundary.
