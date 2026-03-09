---
date: 2026-02-25
source: manual-analysis
issue: ""
tags: [architecture, testing, patterns, debugging]
blog_worthy: true
---

# Why We Reordered the Agent Code Review Pipeline (And Nearly Didn't)

## What Happened

Our autonomous coding agents (Claude Code) were shipping PRs that got rejected by the human reviewer. Analysis of 24 PRs over 5 days revealed: 83% merge rate, 12.5% abandon rate, 8.3% rejection rate. Both rejections (PRs #17 and #32) shared the same root cause — the agent didn't test its own work before declaring "done." We had a proposed pipeline reorder sitting in MEMORY.md for days, marked "implementation pending," but the rationale was based on an unverified claim.

## The Research

### PR Pipeline Data (24 PRs, Feb 20-25 2026)
- **20 merged** (83%), **3 abandoned** (12.5%), **2 rejected** with `changes-requested` (8.3%), **1 open**
- **Rejection pattern**: PR #17 needed 4 revision rounds ("test that it works"). PR #32 rejected for not testing mobile/tablet.
- **Abandon pattern**: PRs #4, #6, #19 — all from day 1, before review process existed. Agent built wrong thing, restarted.
- **No PR has GitHub Review API reviews** — all review happens via labels and comments.

### Living Rules Analysis (22 rules in 5 days)
Distribution by failure category:
- **Playwright/testing anti-patterns**: 4 rules (18%) — trial-and-error learning
- **Schengen calc bugs**: 3 rules (14%) — date handling, missing error checks
- **News-feed integration**: 3 rules (14%) — bad URLs, wrong RSS format, sorting bugs
- **Refactor gotchas**: 2 rules (9%) — knip false positives, type export dependencies
- **UI/a11y**: 2 rules (9%) — loading states, focus traps
- **API integration**: 1 rule — unverified API tier
- **Architecture/docs**: 2 rules — drift, file targeting
- **Misc**: 5 rules — caching, dedup, agent isolation, meta-verification

Key finding: **0 rules came from /judge catching semantic bugs**. All 22 rules came from /reflect self-assessment or direct trial-and-error.

### /reflect Output Completeness
- **Living Rules**: ✅ 22 written, actively used — WORKING
- **Test skeletons**: ❌ 0 written — `~/ai-journal/learnings/` was empty
- **Journal entries**: ❌ 0 written — learnings dir had only .gitkeep
- **Refactor log**: ❌ doesn't exist — `docs/refactor-log.md` never created
- **Implementation score**: 25% (1/4 outputs actually produced)

### Token Economics
- OpenClaw: $1.24/day (budget $120/mo)
- Claude Code: $0.45/day (budget $50/mo)
- Total: $1.69/day, $124.9/mo projected
- **agent-task-runner waste**: 36 runs/night × ~600 tokens × mostly no-ops = ~21,600 tokens wasted per night with 0 issues

### The Unverified Claim
MEMORY.md stated: "PR #73 blocking bug passed tsc/build/E2E but caught by /judge (client-server contract mismatch)." Checked PR #73 — zero comments, zero review history. This claim had no primary evidence. The entire pipeline reorder rationale rested on a confidence game (Judge FM7).

## The Decision

Implemented the reorder anyway, because the data supports it for different reasons:

1. **Verify step addresses the #1 rejection cause** — both rejected PRs failed because the agent didn't test its own work
2. **/reflect before review means the reviewer sees self-corrected code** — even though /reflect's catch rate for semantic bugs is unproven, it produces Living Rules that prevent repeat mistakes across sessions (22 rules in 5 days proves this works)
3. **/refactor before review reduces noise** — reviewer spends time on semantic issues, not dead code
4. **Reviewer agent kept** — different model context provides genuine independence that same-session /judge can't replicate
5. **Auto-safe mode for /refactor** — unblocks autonomous 3am pipeline runs that would otherwise hang on approval prompt

## Risks Acknowledged
- /reflect adds ~30-50K tokens per issue (20 questions + 4 artifacts + file re-reads)
- /refactor adds ~20-30K tokens per issue (6 scans + categorization)
- If /judge triggers fix loop, those tokens are sunk cost from steps that ran on pre-fix code
- Estimated 2-2.5x cost increase per issue if fix loops are common
- Mitigation: daily Optimisation Report cron tracks waste and flags if cost/issue exceeds threshold

## Changes Made (8 files)
1. `~/.claude/agents/backend-engineer.md` — Replaced "Code Review Process" with 6-step pipeline
2. `~/.claude/agents/reviewer.md` — Updated workflow to note code is pre-cleaned by /reflect + /refactor
3. `~/.claude/skills/ship/SKILL.md` — Removed /reflect from Phase 4 (now runs earlier), updated canonical pipeline
4. `~/.claude/skills/refactor/SKILL.md` — Added auto-safe mode for autonomous runs
5. `obos/CLAUDE.md` — Replaced Reflect/Refactor sections with "Implementation Pipeline" section
6. `TOOLS.md` — Updated pipeline description and usage examples
7. `MEMORY.md` — Updated pipeline docs, marked as implemented
8. `agent-task-runner` cron — Updated prompt with explicit 6-step pipeline instructions

## By The Numbers
- 24 PRs analyzed across 5 days
- 22 Living Rules categorized by failure type
- 4 /reflect outputs audited (25% implementation rate)
- 8 files modified to propagate pipeline change
- 1 new cron job created (Optimisation Report, daily 6am Paris)
- 0 verified /judge catches (the PR #73 claim was a confidence game)

## Pattern
**Verify-before-review over review-as-verification**: The cheapest bug to fix is one the author catches before anyone else sees it. Adding a mandatory "run your own code" step before any review is higher-ROI than sophisticated adversarial review on untested code.

**Evidence over narrative**: The proposed pipeline change sat unimplemented because the rationale was a single unverified claim. It was implemented for different, data-backed reasons. Always trace claims to primary evidence before building systems on them.

**Audit the auditor**: /reflect was assumed to be working. Checking its actual output completeness revealed 75% of its mandated artifacts were never produced. Monitoring implementation of quality processes is as important as the processes themselves.
