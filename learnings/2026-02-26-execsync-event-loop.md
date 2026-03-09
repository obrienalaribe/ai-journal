---
date: 2026-02-26
source: reflect
issue: "#76"
tags: [debugging, performance, architecture, testing]
blog_worthy: true
---

# One execSync Blocked 68 Tests

## What Happened
Two E2E tests consistently failed and one was flaky. Screenshots showed pages stuck on "Loading..." spinners. Root cause: three Express request handlers used `execSync` (up to 30s timeout), blocking Node.js's single-threaded event loop and queuing all concurrent HTTP requests. Replacing with async `exec` (promisified) fixed all three test failures.

## The Insight
`execSync` in a Node.js request handler is an invisible time bomb. It works fine in manual testing (one request at a time) but causes cascade failures under concurrent load. The browser fires 3-4 API calls simultaneously when a page loads; if one handler blocks the event loop, all others queue. The fix is always `const execAsync = promisify(exec)` and `await execAsync(...)`.

## By The Numbers
- 2 unexpected failures + 1 flaky test → 69/69 passing (0 failures, 0 flaky) across 3 consecutive runs
- 6 endpoints fixed across 2 rounds: aave (15s), costs (30s), gitlog (5s), getPm2Processes (5s), getHostname (3s), worktree cleanup (30s)
- Key insight: first round fixed 3 obvious blockers but missed 3 subtler ones in `/api/github/prs` path

## Pattern
Async-first child processes in request handlers — never `execSync`, always `promisify(exec)`.
