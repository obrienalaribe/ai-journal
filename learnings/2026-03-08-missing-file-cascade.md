---
date: 2026-03-08
source: reflect
issue: "#307"
tags: [debugging, build, testing]
blog_worthy: false
---

# One Missing File Can Fail Every Test

## What Happened
Five E2E tests on the Usage & Costs page failed with "text not found" errors across three unrelated sections. Investigation revealed the actual cause was a missing component file (`ScriptArchitecturePanel.tsx`) in a completely different page (Content Engine). The TypeScript build failed, producing no `dist/` output, which meant Playwright's web server had nothing to serve.

## The Insight
When E2E tests fail across multiple unrelated pages or sections simultaneously, the root cause is almost never in the individual pages — it's a build-level failure. Always run `npm run build` before investigating individual component rendering logic.

## By The Numbers
- 5 tests failing -> 5 tests passing (+ 33 regression tests verified)
- 1 file created (151 lines) to fix all 5 failures

## Pattern
Build-first diagnosis: global E2E failure = check build before investigating individual components.
