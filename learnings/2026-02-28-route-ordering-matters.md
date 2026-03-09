---
date: 2026-02-28
source: reflect
issue: "#177"
tags: [architecture, debugging, patterns]
blog_worthy: false
---

# Express Parameterized Routes Silently Shadow Literal Routes

## What Happened
Added a `/schedule/queue` endpoint alongside an existing `/schedule/:week` route. The parameterized route matched "queue" as a week string, returning week-format data instead of queue data. Discovered via curl verification — the build and type checker couldn't catch this.

## The Insight
Express route matching is order-dependent and `:param` matches any string. When adding literal routes alongside parameterized ones, registration order is the only defense. curl verification after implementation catches routing bugs that static analysis cannot.

## By The Numbers
- 1 route ordering bug caught by manual curl verification
- 5 files modified across 3 layers (backend, frontend, workflow)

## Pattern
Literal-before-parameterized Express route registration
