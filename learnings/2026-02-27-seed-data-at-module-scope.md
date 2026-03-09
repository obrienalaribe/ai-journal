---
date: 2026-02-27
source: reflect
issue: "#146"
tags: [architecture, patterns]
blog_worthy: false
---

# Seed Configuration Data at Module Load, Not Request Time

## What Happened
Building a keyword editor for event scoring required default keyword data to exist for 9 categories. Seeding at import time with an `existsSync` guard ensures the file is written once at server startup and never overwrites user edits. The alternative — seeding on first GET request — would race under concurrent requests.

## The Insight
Config seed data belongs at module scope with idempotent guards. Request handlers should assume config exists (or gracefully handle absence), never create it as a side effect.

## By The Numbers
- 9 category keyword configs seeded (109 keywords total)
- 0 request-time writes needed after startup

## Pattern
Module-scope idempotent seed: `if (!existsSync(path)) writeFileSync(path, defaults)`
