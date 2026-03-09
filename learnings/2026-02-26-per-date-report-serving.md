---
date: 2026-02-26
source: reflect
issue: "#78"
tags: [architecture, testing, patterns]
blog_worthy: false
---

# Static assets need date-stamped archiving when dashboards show historical data

## What Happened
A dashboard showed E2E test results per date (68 passed, 2 failed) but the "VIEW REPORT" link always opened the latest Playwright HTML report, not the one matching that date's card. Fixed by archiving HTML reports to dated subdirectories and adding per-date Express routes with fallback redirect.

## The Insight
When a UI displays time-series data with links to associated artifacts (reports, logs, builds), the artifact storage must match the data granularity. A single overwritten file creates a data-artifact mismatch that erodes trust in the dashboard.

## By The Numbers
- 3 files changed, ~45 lines added
- 0 regressions: 69 tests pass, 1 pre-existing failure (unmerged normalizer)

## Pattern
Date-stamped artifact archiving: process script writes both structured data AND copies the artifact to a dated directory in one atomic step.
