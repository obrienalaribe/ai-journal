---
date: 2026-03-09
source: reflect
issue: "memory-hygiene"
tags: [architecture, ai-agents, memory, patterns]
blog_worthy: true
---

# Progressive disclosure for AI agent memory

## What Happened
MEMORY.md grew to 38K chars / 220 lines over 3 weeks of intensive development. The system loads only the first 200 lines into every session context. Result: 45% of accumulated knowledge was silently truncated. The agent would re-learn documented patterns, waste context re-discovering file paths, and occasionally contradict its own prior conclusions.

Nobody noticed because truncation doesn't error — the agent just operates with partial memory and compensates by re-reading files.

## The 3-Layer Solution

- **Layer 1: Always-loaded index** (`MEMORY.md` — ~70 lines). Cross-cutting facts: architecture, key files, build commands, API patterns. Fits within the 200-line system load. Links to deeper layers.
- **Layer 2: Path-triggered rules** (`.claude/rules/*.md` — 11 files). Auto-injected when the agent touches matching file paths. Schengen rules load only when editing Schengen files. Zero cost when inactive.
- **Layer 3: On-demand topics** (`memory/topics/*.md` — 7 files). Deep knowledge (content pipeline, delivery, data schemas, n8n, usage/costs, skills, blog). Agent reads these when needed, not every session.

## By The Numbers
- MEMORY.md: 38K → 5.4K chars (86% reduction)
- CLAUDE.md Living Rules: 115 → 8 rules (93% reduction, rest moved to rules/)
- Topic files created: 7
- Rules files: 11 (path-scoped, auto-injected)
- Knowledge lost: 0 (verified by grep audit)

## The Insight
Information that *might be needed* costs context window space every session. Information that's *indexed and discoverable* costs nothing until needed. This is the same progressive disclosure principle used in documentation for humans — show the summary, link to the detail, load on demand.

## Pattern
Three-layer progressive disclosure: always-loaded index → auto-triggered rules → on-demand deep reference. Applicable to any AI agent with persistent memory and limited context windows.
