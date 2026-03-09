---
date: 2026-02-25
source: manual-analysis
issue: ""
tags: [architecture, testing, patterns]
blog_worthy: true
---

# Why Your AI Coding Agents Need a Homework Checker

## What Happened

We built a 7-step implementation pipeline for autonomous coding agents: implement → verify → self-reflect → refactor → adversarial review → fix loop → ship. The self-reflection step (/reflect) mandates 4 artifacts: Living Rules, test skeletons, journal entries, and memory corrections. Auditing actual output revealed only 25% completion — Living Rules were written (22 in 5 days), but test skeletons, journal entries, and refactor logs were never produced. The builder agent was skipping 3 of 4 mandated outputs and nobody caught it.

## The Insight

Self-assessment by the same agent that did the work has a fundamental blind spot: the agent wants to move on to the next step. It writes the minimum viable reflection (Living Rules are quick to append) and skips the expensive artifacts (test skeletons require reading Playwright patterns, journal entries require structured writing). A separate agent with a single job — verify the paperwork exists — closes this gap at minimal cost (~2K tokens per audit).

## By The Numbers
- /reflect artifact completion before auditor: 25% (1/4 outputs produced)
- Living Rules written: 22 across 5 days (this output works)
- Test skeletons written: 0 (never produced)
- Journal entries written: 0 (never produced)
- Refactor log entries: 0 (file never created)
- Estimated auditor cost: ~$0.01-0.02 per run (sonnet, small context)
- Files the auditor reads: 6 locations
- Files the auditor writes: 1 (audit report JSON)

## The Agent Roster (4 agents, 7-step pipeline)
| Agent | Model | Role | Why Separate Agent? |
|-------|-------|------|---------------------|
| backend-engineer | opus | Build + verify + reflect + refactor | Needs full implementation context |
| quality-auditor | sonnet | Verify artifacts exist + meet quality bar | Different incentive: not the builder, checks homework objectively |
| reviewer | sonnet | Adversarial /judge review | Different model = different blind spots than builder |
| ai-accelerator | opus | Product research + ideation | Creative divergent thinking, separate from engineering |

## Pipeline Evolution
```
v1 (Feb 20): implement → reviewer → ship
v2 (Feb 23): implement → reviewer → ship → reflect (post-push)
v3 (Feb 25 AM): implement + verify → reflect → refactor → judge → ship
v4 (Feb 25 PM): implement + verify → reflect → refactor → quality-auditor → judge → ship
```

Each version addressed a specific measured failure:
- v1→v2: No learning capture → added /reflect
- v2→v3: /reflect ran too late (post-push) + agent didn't test own work → moved reflect earlier + added verify
- v3→v4: /reflect skipping 75% of outputs → added quality-auditor to enforce completion

## Upcoming: Content Intelligence Agent
Proposed (Issue #74): Separate agent for creative content skills (/reframe, /intelligence, /story-match, /hook-engineer, /audience-pulse). Engineering agents optimize for correctness. Content needs contrarian takes and provocative framing — fundamentally different cognitive mode.

## Pattern
**Separate the incentives**: The agent that builds wants to ship. The agent that audits wants completeness. The agent that reviews wants correctness. Mixing these in one agent creates conflicts of interest — the builder will always shortcut the audit to move faster. Dedicated agents with single responsibilities and different models produce better outcomes than one agent doing everything.

**The homework checker principle**: If a process mandates outputs, add a cheap verification step with a different agent. The cost of checking (~$0.01) is orders of magnitude less than the cost of missing artifacts accumulating (0 test skeletons across 22 sessions = 22 unguarded regressions).
