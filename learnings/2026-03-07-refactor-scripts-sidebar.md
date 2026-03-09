---
date: 2026-03-07
source: refactor
issue: "#295"
tags: [dead-code, file-size]
blog_worthy: false
---

# Scripts Tab refactor: zero new dead code from +286 line sidebar addition

## What Was Found
Scans detected 5 pre-existing dead code items (unused files, exports, types) and 15 low-severity clones. The new ScriptsTab grew from 890 to 1176 LOC with the intelligence sidebar but introduced zero new dead code or duplication. One CSS style selector anti-pattern was caught and removed from tests.

## What Was Removed
- 1 test anti-pattern: `[style*='border-radius: 6px']` replaced with semantic locator

## By The Numbers
- ScriptsTab.tsx: 890 → 1176 LOC (+32% for full sidebar panel)
- Test anti-patterns in new code: 1 → 0
- New dead code introduced: 0

## Pattern
Inline scan during implementation catches anti-patterns before they embed
