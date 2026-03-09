---
date: 2026-02-26
source: reflect
issue: "#87"
tags: [architecture, patterns]
blog_worthy: false
---

# File-based contracts beat API coupling for local tool integration

## What Happened
Google Calendar ICS exports only show "Busy" — hiding event details. Instead of building a direct Google API integration into the Express server, an n8n workflow writes structured JSON to a shared filesystem path. The server reads it on demand with zero coupling to Google's API.

## The Insight
When integrating local automation tools (n8n, cron, scripts) with a dashboard server, file-based JSON contracts are simpler and more reliable than HTTP APIs between services. The file is the contract — the producer owns the shape, the consumer just reads.

## By The Numbers
- 15 lines of server.js added (vs ~100+ for a direct Google Calendar API integration)
- 2 files modified total

## Pattern
File-contract integration: automation tool writes JSON → server reads on request → graceful fallback if absent
