---
date: 2026-03-09
source: reflect
issue: "#334"
tags: [testing, patterns, ci]
blog_worthy: false
---

# Don't Seed Test Data via Filesystem When Server Path Differs

## What Happened
E2E test for brief evaluation scores seeded a JSON file via `fs.writeFileSync(process.cwd() + "/../data/...")`. When tests ran against a remote server (different directory), the server couldn't find the file. Changed to API discovery pattern: find an existing score via the API, validate its shape.

## The Insight
Tests that seed data via filesystem are tightly coupled to the server's directory layout. When CI or worktrees run the server from a different path, seeded files become invisible. API-based discovery or API-based seeding is always more portable.

## By The Numbers
- 4 E2E failures fixed in one revision (0 failures after)
- 36 content tests passing, up from 32

## Pattern
API-discovery over filesystem-seeding for E2E test data
