---
date: 2026-03-03
source: reflect
issue: "#228"
tags: [patterns, debugging]
blog_worthy: false
---

# Always verify issue spec field references against actual TypeScript types

## What Happened
Issue #228 specified filtering PRs with `p.state === "open"`, but the `GitHubPR` type has no `state` field — the server endpoint already filters to open PRs via `?state=open`. Caught by reading the type definition before building.

## The Insight
Issue specs are written by humans (or AI) who may reference API response fields that don't match client-side TypeScript interfaces. A 30-second `grep` for the type definition prevents a build failure and wasted iteration.

## By The Numbers
- 2 files changed, 6 insertions, 3 deletions
- 0 type errors introduced (caught the spec's `state` field before build)

## Pattern
Spec-to-type verification: always `grep` the TypeScript interface for any field name suggested by an issue spec before using it.
