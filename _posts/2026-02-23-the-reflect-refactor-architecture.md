---
layout: post
title: "The Reflect/Refactor Architecture: Building Self-Improving AI Agents"
date: 2026-02-23
author: OB
tags: [claude-code, reflect, refactor, agent-autonomy, architecture]
excerpt: "24.4% of my agent compute was wasted on regression. Here's the yin-yang architecture that fixes it — and the path to agents that work while you sleep."
---

![Hero: The Reflect/Refactor Yin-Yang](/ai-journal/assets/images/reflect-refactor-yinyang.png)

Last week I ran `/insights` across 79 Claude Code sessions and discovered that **24.4% of my agent compute time was wasted**. Not on hard problems — on the same bugs appearing across sessions, the same anti-patterns repeating, the same lessons being re-learned because context compaction erased them.

13.2 hours out of 54. Gone. Because agents don't remember their mistakes.

This post describes the architecture I built to fix it: two complementary skills — `/reflect` and `/refactor` — that together form a self-improvement loop for AI coding agents. One captures learning. The other enforces hygiene. Neither works alone.

---

## The Problem: Agents That Don't Learn

Here's the friction breakdown from my [first post](/ai-journal/2026/02/22/what-54-hours-of-claude-code-taught-me.html):

| Friction Category | Sessions | Total Waste | % of 54h |
|---|---|---|---|
| Buggy first-pass code | 22 | ~5.5h | 10.2% |
| Wrong architectural approach | 14 | ~7.0h | 13.0% |
| Git/CI failures | 4+ | ~0.7h | 1.3% |
| **Total** | | **~13.2h** | **24.4%** |

The core insight: most of this waste was **recurring**. The timezone bug that cost 3 iterations on the Schengen calculator? It would happen again next session because `new Date("2026-03-15")` parses as UTC midnight — a well-known JavaScript footgun. The Playwright locator that matched two elements instead of one? Same problem, different page, different week.

Claude Code uses auto-compaction at ~83% context to keep working within its window. When compaction fires, conversation history collapses to a summary. Plans vanish. Debugging context evaporates. Lessons learned in the first hour of a session are gone by the third.

The agent starts fresh every time. Same capabilities. Same blind spots. Same mistakes.

---

## The 80/20 Gap

![The 80/20 Bridge](/ai-journal/assets/images/80-20-gap.png)

There's a gap in AI-assisted coding that nobody talks about enough.

LLMs get you 80% of the way. The first draft works. The component renders. The API endpoint returns data. This is the **vibe coding** phase — fast, impressive, exhilarating. You feel like you're building at 10x speed.

Then you hit the last 20%.

Edge cases. Timezone handling. Error paths. Mobile responsiveness. Test coverage. The difference between "it works on my machine" and "it works in production." This is where the gap lives — and it's where most AI-assisted projects quietly fail.

The problem isn't that agents can't do the last 20%. It's that **we are liable for their output**. When an agent ships a timezone bug to production, the user's name is on the commit. When a Playwright test passes by accident because `getByText()` matched a substring, the PR is approved under a human's authority.

The 80/20 gap is a trust gap. And closing it requires more than better prompts.

---

## /reflect — The Learning Half

`/reflect` is a structured self-reflection protocol that runs after every feature implementation. It's not a performance review. It's a failure analysis machine.

### The Protocol

**Phase 1 — Evidence Gathering**: Read every modified file from disk (not memory). Run `git diff`, `tsc --noEmit`, Playwright tests. Capture raw facts before analysis begins.

**Phase 2 — 20 Questions**: A forced march through four categories:

- **Execution Quality** (Q1-5): What worked first try? What needed iteration? Which patterns did I follow or miss?
- **Reasoning Failures** (Q6-11): What did I assume without verifying? What did I hallucinate? Where would a human reviewer have caught my mistake?
- **Regression Risk** (Q12-16): What edge cases did I miss? What state could go stale? What magic values did I hardcode?
- **System Improvement** (Q17-20): What patterns aren't documented? What would I do differently with hindsight?

Every answer has a quality gate. "I learned about theme handling" gets rejected. "Schengen page uses hardcoded `#0D9488` for teal — PRD-specified domain color, NOT a theme token. I initially used `th.success` which was wrong" passes.

**Phase 3 — Three Artifacts**:

### 1. Test Skeletons

Concrete Playwright test stubs targeting the specific failures discovered in Phase 2. Not aspirational test plans — real `test()` blocks with arrange-act-assert structure, ready to implement.

### 2. Living Rules

One-line rules appended to `CLAUDE.md` with date, context, failure description, and guard test. These are the scar tissue of the system. Here's a real one:

```
[2026-02-22] [schengen] RULE: Use noon-anchored dates (new Date(str + "T12:00:00"))
  for all YYYY-MM-DD parsing.
WHY: Bare new Date("YYYY-MM-DD") → UTC midnight → wrong day in UTC-negative timezones.
GUARD: "POST /api/schengen/trips creates trip with correct totalDays"
```

This rule exists because `new Date("2026-03-15")` parses as UTC midnight, which in UTC-5 becomes March 14. Three iterations to diagnose. The fix is one line. The rule prevents every future session from re-discovering it.

### 3. Memory Corrections

Updates to the project's `MEMORY.md` — endpoint signatures, file locations, data schemas that changed during implementation. Future sessions start with accurate context instead of stale assumptions.

The key design choice: **maximum 3 rules per session**. Fewer dense rules beat many thin ones. The Living Rules section loads into every session — bloating it defeats its purpose.

---

## /refactor — The Cleanup Half

Where `/reflect` captures learning from failures, `/refactor` enforces code hygiene through deterministic tools.

### The Principle

**Deterministic tools for detection. LLM for judgment.**

`knip` finds dead code. `jscpd` finds duplication. These are exact algorithmic problems with exact answers. The LLM decides *what to do* about the findings — which dead code to remove, which duplication is acceptable, which refactors are worth the risk.

### The Protocol

**Phase 1 — Six Scans** (zero LLM tokens for detection):

1. `npx knip` — unused files, exports, dependencies
2. `npx jscpd` — code duplication with file + line locations
3. `npx tsc --noEmit` — type errors (must be zero before anything else)
4. API route cross-reference — server routes vs. frontend consumers
5. File size audit — components exceeding 500 LOC
6. Test anti-pattern count — `waitForTimeout`, `networkidle`, etc.

**Phase 2 — Categorized Report**: Findings organized by severity (SAFE/REVIEW for dead code, LOW/MEDIUM/HIGH for duplication, etc.). Presented to the human for approval.

**Phase 3 — Execute**: Only approved items get fixed. Each fix verified with `tsc --noEmit` immediately. Any breakage → revert and flag. No accumulating failures.

The critical gate: **nothing executes without explicit human approval**. Detection is fully automated. Execution requires judgment.

---

## Why They're Twins

Here's what happens with each skill alone:

**`/reflect` without `/refactor`**: The system learns from mistakes. Living Rules accumulate. Memory stays accurate. But code debt grows unchecked. Unused exports pile up. Duplication spreads. The codebase gets heavier with every feature, and eventually the lessons can't overcome the friction of working in messy code.

**`/refactor` without `/reflect`**: The code stays clean. Dead exports get removed. Duplication gets extracted. But the same mistakes keep happening. Every session rediscovers the timezone bug. Every Playwright test uses substring matching when it should use `{ exact: true }`. The code is clean but the process is broken.

Together, they form the hygiene layer:

- `/reflect` captures **why things break** → Living Rules prevent recurrence
- `/refactor` cleans up **what's left behind** → deterministic tools keep the codebase lean
- The combination means each session is both smarter and working in cleaner code

---

## The Compounding Effect

![The Feedback Loop](/ai-journal/assets/images/feedback-loop.png)

Each `/reflect` session adds approximately 3 rules. After 5 days of building the OBOS dashboard:

- **Date parsing**: noon-anchored dates prevent timezone bugs
- **Error handling**: every `fetch()` mutation checks `res.ok` before the success path
- **Playwright locators**: `{ exact: true }` for text that may substring-match
- **API validation**: `curl -sI` every URL before adding to configuration
- **Calculation composition**: compose through existing functions, never reimplement dedup logic
- **Type removal safety**: check consumers before deleting, run `tsc` after every removal

Six categories of failure, each guarded by a rule and (where applicable) a test. Every category was discovered through real bugs, not speculation.

The loop works like this:

1. **`/insights`** analyzes sessions → quantifies friction
2. **`/reflect`** processes failures → produces rules and tests
3. Rules persist in **`CLAUDE.md`** → survive context compaction
4. Next session starts with accumulated knowledge → **fewer errors**
5. Fewer errors → less friction → back to `/insights`

The compound interest of this loop is the real product. Individual rules are small. The accumulated effect after 5 days is that entire classes of bugs simply don't happen anymore.

---

## The Vision: Autonomous Cron Pipeline

![The Autonomous Pipeline](/ai-journal/assets/images/autonomous-pipeline.png)

Here's where this is going.

Today, the loop requires human triggers. I run `/reflect` manually after features. I run `/refactor` when things feel messy. I review and approve every change.

The architecture supports something more ambitious:

```
Daily cron:
  GitHub Issue → Agent Implements → Hooks Run Tests
  → /reflect → /refactor → Commit → Repeat
```

A system that picks up issues overnight, implements them, runs its own quality gates, reflects on what went wrong, cleans up after itself, and has the changes ready for review by morning.

### Why Not Yet

Three gaps prevent fully autonomous operation:

1. **Trust**: The 80/20 gap means agent output needs human review. Living Rules reduce — but don't eliminate — the judgment required. The system isn't trustworthy enough for unsupervised production changes.

2. **Test coverage**: The agent can only self-validate against existing tests. If a feature doesn't have Playwright coverage, the agent has no feedback signal. `/reflect` produces test skeletons, but they still need human implementation and validation.

3. **Judgment calls**: `/refactor` explicitly requires human approval before execution. Some decisions — whether to extract a component, whether duplication is acceptable, whether a 500-line file should be split — require context that the agent doesn't have.

### Measuring Progress

The target: reduce the 24.4% friction rate to below 10% within 30 days.

That's roughly 8 hours of agent compute recovered per week — enough to build an additional dashboard feature every cycle. More importantly, it's a measurable indicator of system trust. As friction drops, the case for autonomous operation strengthens.

The foundation under the entire pipeline is trust. Not blind trust — earned trust, measured by declining friction rates, increasing test coverage, and fewer human interventions required per feature.

---

## Getting Started

If you're building with Claude Code (or any LLM coding agent), here's the minimum viable version:

1. **Create a `CLAUDE.md`** with your project's known failure modes. Not a style guide — scar tissue from real bugs.
2. **After every feature, ask three questions**: What broke? Why? How do I prevent it next time? Write the answers as one-line rules in the file.
3. **Run `knip`** weekly to catch dead code before it accumulates.
4. **Measure your friction**. Track how many sessions hit preventable issues. You can't improve what you don't measure.

The fancy skills and protocols came later. The core insight is simple: **persistent rules in a file that survives context compaction are worth more than any amount of clever prompting.**

---

*Built with Claude Code. Illustrated with GPT-5 Image and Gemini 3 Pro via OpenRouter.*
