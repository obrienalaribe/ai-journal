---
date: 2026-02-27
source: reflect
issue: "#104"
tags: [testing, ci, debugging]
blog_worthy: false
---

# Your Test Runner's Timeout Includes Your Build Time

## What Happened
Nightly E2E tests started failing with "webServer process exited with code 1." The Playwright webServer command ran `node server.js`, which auto-builds the frontend if `dist/` is missing. The build takes ~19 seconds, but the webServer timeout was 15 seconds. Fixed by prepending `npm run build &&` to the command and increasing timeout to 60s.

## The Insight
When your test framework manages a dev server, its startup timeout covers everything from command invocation to URL responding — including any build steps. Separate build from serve in your test command so timeouts are predictable.

## By The Numbers
- Build time: ~19s (exceeds 15s timeout)
- New timeout: 60s (3x headroom)
- Fix size: 2 lines changed in 1 file

## Pattern
Separate build-then-serve over auto-build-on-startup in test contexts
