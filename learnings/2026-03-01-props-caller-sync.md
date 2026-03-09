---
date: 2026-03-01
source: reflect
issue: "#203"
tags: [typescript, patterns]
blog_worthy: false
---

# When removing React props, update the signature AND every callsite atomically

## What Happened
IdeaCard's action buttons were moved to IdeaDetailModal. The prior agent removed `onRefresh` and `viewFilter` from the component's type signature but left them in the JSX callsite. TypeScript caught this instantly via `tsc --noEmit`.

## The Insight
Prop removals in TypeScript components require two synchronized edits: the type signature and every `<Component prop={...}>` callsite. Grep for `<ComponentName` before committing to find all callers.

## By The Numbers
- 1 TS error caught by `tsc --noEmit`
- 2-line fix (remove 2 prop passes from JSX)

## Pattern
Atomic prop removal: grep callsites before committing type signature changes
