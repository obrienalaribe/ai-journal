---
date: 2026-03-08
source: refactor
issue: "#318"
tags: [dead-code, architecture]
blog_worthy: false
---

# Over-exported symbols accumulate silently during module extraction

## What Was Found
After splitting a 3976-line monolith into 4 modules, knip detected 8 symbols exported from the shared helpers module that no external consumer imports. All 8 were used internally — the `export` keyword was carried over from the original monolith where everything lived in one scope.

## What Was Removed
- 7 `export` keywords removed from internal-only constants and functions
- 1 unused re-export (`PROJECT_ROOT_DIR`) removed from the barrel export line

## By The Numbers
- knip content-helpers.js findings: 8 → 0
- Total route count preserved: 81 (40 + 8 + 24 + 9)
- content.js: 3976 → 1099 lines

## Pattern
Run knip immediately after any module extraction — over-exports are the #1 hygiene issue in split refactors
