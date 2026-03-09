---
date: 2026-02-27
source: reflect
issue: "#125"
tags: [testing, patterns, architecture]
blog_worthy: false
---

# Replacing visual components silently breaks E2E tests on output format

## What Happened
Replaced a score circle (showed integer 1-10) with a donut chart (shows weighted decimal like 7.4) in the Ideas tab. Build and TypeScript passed clean. The /reflect phase caught that an existing E2E test regex `/^\d{1,2}$/` would fail because it expects integer-only output.

## The Insight
When upgrading a visual component's data representation (integer to decimal, checkmark to progress bar), the build pipeline won't catch E2E regressions on rendered text format. A grep for the old component name in test files should be standard practice before removing it.

## By The Numbers
- 3 files changed, 366 insertions, 96 deletions
- 1 E2E test identified as at-risk before it could fail in CI

## Pattern
Pre-removal test audit: `grep -r "OldComponentName\|old output text" tests/` before deleting
