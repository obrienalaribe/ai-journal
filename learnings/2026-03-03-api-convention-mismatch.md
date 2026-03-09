---
date: 2026-03-03
source: reflect
issue: "#225"
tags: [hooks, patterns, dx]
blog_worthy: false
---

# Custom hooks don't follow library conventions — verify before use

## What Happened
While building a photo upload UI, I destructured `mutate` from a `useApi()` hook — a pattern from SWR/React Query that doesn't exist in this custom hook. TypeScript caught it immediately, but it was a clear case of library convention assumptions leaking into custom code.

## The Insight
Custom hooks rarely match the API of popular libraries they're inspired by. The 10 seconds it takes to read a hook's return type saves minutes of debugging type errors. This is especially true in codebases where hooks are 30-50 lines — just read the whole file.

## By The Numbers
- 1 tsc error caught (unused `mutate` destructure)
- 51 lines in `useApi.ts` — trivial to read upfront

## Pattern
Verify-hook-signature-before-destructure — `grep "return {" src/hooks/useXxx.ts` before first use
