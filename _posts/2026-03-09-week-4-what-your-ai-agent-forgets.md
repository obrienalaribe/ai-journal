---
layout: post
title: "Week 4: What Your AI Agent Forgets Every Session"
date: 2026-03-09
author: OB
tags: [claude-code, weekly, memory, observability, progressive-disclosure]
excerpt: "89 commits, 4 blog-worthy learnings, and the discovery that my AI agent was silently losing 45% of its accumulated knowledge every session."
---

## This Week in Numbers

| Metric | Value |
|--------|-------|
| Commits | 89 |
| Features shipped | ~15 (evidence pipeline, route split, memory hygiene, voice eval) |
| Bugs fixed | ~12 (E2E stability, namespace collision, brief format) |
| Living Rules added | 3 (evidence, architecture, git) |
| Blog-worthy learnings | 4 |
| E2E tests | 262 total |
| MEMORY.md reduction | 86% (38K → 5.4K chars) |

Week 4 was a velocity week. Evidence pipeline, route monolith split, voice evaluation gate, Dark Factory architecture. But the most important discovery wasn't a feature — it was a silent failure. My AI agent had been forgetting nearly half its accumulated knowledge every session, and neither of us noticed until I counted the characters.

## What We Built

**Evidence pipeline (DOM → API → server).** The judge agent previously made approval decisions based on 2 evidence types: before and after screenshots. This week we added interaction screenshots, API call interception logs, and QA reports — 5 evidence types total. More importantly, the pipeline moved from step 6 (post-approval) to step 5 (pre-approval). The agent was making decisions blind. Now it sees everything before rendering a verdict.

**Route monolith split.** `routes/content.js` hit 3,976 lines — the largest file in the codebase. Split into 4 focused modules: `content.js` (1,099), `photos.js` (217), `schedule.js` (1,511), plus `content-helpers.js` (723) for shared constants and utilities. 81 API routes preserved, 0 path regressions. The key insight: always audit the dependency graph before splitting. "Duplicate everything" looks simpler but creates constant-drift bugs.

**Voice evaluation gate.** LLM-scored evaluation cards for PSL briefs (hook/setup/story format). Structured scoring rubric, regeneration pass when scores fall below threshold, Scripts tab UI for reviewing and approving voice-evaluated content.

**Memory hygiene.** The three-layer rules architecture described below. Plus: CLAUDE.md went from 115 inline Living Rules to 8 cross-cutting rules, with the rest distributed to 11 path-scoped rule files that auto-inject when relevant.

## The Discovery: Your Agent Is Forgetting

Three weeks into the project, I had a well-populated `MEMORY.md` file. 220 lines of architecture knowledge, file paths, API patterns, build commands, feature-specific conventions. The kind of institutional knowledge that makes an AI agent feel like a colleague rather than a stranger.

There was one problem: the system loads only the first 200 lines.

```
MEMORY.md: 220 lines
System loads: 200 lines
Silently truncated: 20 lines (9%)
```

That doesn't sound bad. But character count tells the real story:

```
Total: 38,000 chars
Loaded: ~21,000 chars
Truncated: ~17,000 chars (45%)
```

The last 20 lines contained the densest information — feature-specific patterns, cross-cutting conventions, hard-won debugging insights. The first 180 lines were architecture basics that the agent could rediscover by reading files. The most valuable knowledge was the most likely to be cut.

The symptom was subtle: the agent would re-learn patterns it had already documented, waste context re-discovering file locations, and occasionally contradict its own prior conclusions. No error. No warning. Just silent degradation.

### The Three-Layer Fix

The solution borrows from the same progressive disclosure principle used in documentation for humans: show the summary, link to the detail, load on demand.

**Layer 1: Always-loaded index** — `MEMORY.md`, trimmed to ~70 lines. Cross-cutting facts only: architecture overview, key files, build commands, API patterns, theme tokens, dashboard page list. Everything that applies to every session. Links to deeper layers.

**Layer 2: Path-triggered rules** — `.claude/rules/*.md`, 11 files. Auto-injected by the system when the agent touches matching file paths. Schengen calculation rules load only when editing Schengen files. LanceDB schema rules load only when touching intelligence routes. Zero context cost when inactive.

**Layer 3: On-demand topics** — `memory/topics/*.md`, 7 files. Deep reference material: content pipeline stages, delivery queue lifecycle, data file schemas, n8n monitoring, usage/costs architecture, skills/agents inventory, blog pipeline. The agent reads these when a task requires it, not every session.

![Pencil sketch: Three horizontal layers stacked vertically. Top layer (solid border): "MEMORY.md — 70 lines" with a brain icon. Middle layer (dashed border): ".claude/rules/ — auto-injected" with a compass icon. Bottom layer (dotted border): "topics/ — on-demand" with a filing cabinet icon. A bracket on the left shows "Context Window" spanning only the top layer. Arrows from the middle and bottom layers are labeled "loaded when needed." Caption: "Progressive disclosure for AI agent memory."](/ai-journal/assets/images/week-4-memory-layers.png)

### The Numbers

| Before | After |
|--------|-------|
| MEMORY.md: 38K chars, 220 lines | MEMORY.md: 5.4K chars, ~70 lines |
| CLAUDE.md: 115 inline Living Rules | CLAUDE.md: 8 cross-cutting rules |
| Rules files: 0 | Rules files: 11 (path-scoped) |
| Topic files: 0 | Topic files: 7 |
| Knowledge lost: — | Knowledge lost: 0 |

86% reduction in always-loaded memory. Zero knowledge lost. The information didn't disappear — it moved to where it's discoverable when needed and invisible when not.

### The Insight

Information that *might be needed* costs context window space every session. Information that's *indexed and discoverable* costs nothing until needed.

This is the same principle that makes Unix man pages work: you don't load every manual page into memory at boot. You know they exist, you know how to find them, and you load them when you need them. AI agent memory should work the same way.

The failure mode to watch for: the memory file grows silently. It's not a file you actively maintain — it accumulates through automated reflection sessions. By the time it hits the truncation limit, the most recent (and most valuable) entries are the ones being cut. A memory system that grows without pruning is a memory system that forgets.

## What Went Wrong

**4 E2E failures from the nightly run (#346–#349).** The nightly maintenance script caught: SAVED tab dismiss action not updating status (ECONNRESET from server restart timing), queue items leaking between test runs (isolation gap), generate-linkedin returning unexpected errors, and intelligence signal badge visibility.

**Namespace collision (#333).** `source_ids` and `pinned_intelligence_ids` were sharing the same field in the data model. Intelligence queries returned wrong documents when an idea had both pinned intelligence items and regular source references. The IDs overlapped silently — both were valid UUIDs, both pointed to real records, but in different collections.

**4/8 AI critique claims hallucinated.** During a LanceDB SDK audit, the adversarial reviewer flagged 8 "improvements." Cross-referencing against actual `.d.ts` type definitions: 4 of the 8 methods didn't exist. The reviewer confidently suggested migrating to APIs that were fabricated. Always validate AI-generated SDK claims against type definitions.

## What Fixed It

**Separate namespace fields (#333).** Split `source_ids` into two explicit fields: `source_ids` for content sources and `pinned_intelligence_ids` for intelligence references. Schema migration, not a code hack.

**Pre-flight health check (#290).** Added a server liveness check before Playwright runs in the nightly maintenance script. ECONNRESET failures were caused by tests starting before the server finished restarting.

**Structured brief format (#329).** Enforced HOOK/SETUP/STORY format for PSL briefs at the generation endpoint — outline structure, not prose. Voice evaluation scores jumped when the LLM had structured input rather than freeform paragraphs.

**Living Rule: export format first.** "When a feature has 'export as X format' + 'render in-app', improve the export format first." This rule prevented 423 lines of custom HTML mind-map renderer. The actual need was 80 lines of improved Excalidraw JSON generation.

## Lessons Learned

1. **Silent truncation is worse than a crash.** MEMORY.md was silently losing 45% of its content and nobody noticed because it didn't error. The agent just operated with partial knowledge and compensated by re-reading files. If your system has limits, make them visible.

2. **Information architecture for AI agents follows the same progressive disclosure principles as documentation for humans.** Show the index. Link to the detail. Load on demand. The context window is a public good — don't fill it with information that might be needed.

3. **Evidence-as-input > evidence-as-afterthought.** Moving screenshots before the review decision (step 6 → step 5) changed the judge's verdict quality. The same data, consumed earlier in the pipeline, produces better decisions. Pipeline ordering matters more than pipeline contents.

4. **The 10% Rule prevented 423 lines of custom renderer** when 80 lines of Excalidraw improvement solved the real need. Always ask: does a custom in-app renderer add value that the export tool cannot provide? If the answer is "convenience," that's not enough.

5. **4/8 AI-generated SDK critique claims were hallucinated.** Always validate against `.d.ts` files. The reviewer was confidently wrong about methods that didn't exist. Trust AI for patterns, verify AI for specifics.

## Living Rules Added This Week

```
[2026-03-07] [evidence] RULE: page.on('response') callbacks are
fire-and-forget async — always try/catch await res.json().
WHY: Playwright event emitter doesn't await async handlers.
Page close before json() completes throws unhandled error.

[2026-03-08] [architecture] RULE: When a feature has "export as X format"
+ "render in-app", improve the export format first.
WHY: Built 423-line HTML renderer when 80 lines of Excalidraw
improvement solved the actual goal.

[2026-03-09] [git] RULE: When rebasing across multiple sibling branches,
use git merge not sequential git rebase — rebase replays intermediate
commits and creates duplicates.
WHY: Sequential rebase across 3 branches duplicated commits
and created unresolvable conflicts.
```

## What's Next

- Fix 4 nightly E2E failures (#346–#349)
- Intelligence Engine bug fixes (#261): hybrid score zeroing, reranker noise
- Dark Factory voice pipeline: ship first real piece of content through the full auto pipeline
- Progressive disclosure measurement: track whether context utilization improves over the next week's sessions

---

*Week 4 of building with autonomous AI agents. 89 commits. The most important discovery wasn't a feature — it was a failure mode hiding in plain sight. Your agent's memory file is probably too long, and it's forgetting the best parts.*

*Generated from git history + /reflect learnings. Reviewed and published by OB.*
