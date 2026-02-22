---
layout: post
title: "What 54 Hours of Claude Code Taught Me About AI Agent Orchestration"
date: 2026-02-22
author: OB
tags: [claude-code, agent-orchestration, insights, obos]
excerpt: "I spent five days building a personal operations dashboard entirely with Claude Code agents. 54 hours, 56 sessions, 24.4% friction rate. Here's what went wrong, quantified, and what fixed it."
---

I spent five days building a personal operations dashboard entirely with Claude Code agents. 12 pages, 44+ API endpoints, 11,500 lines of TypeScript, 52 commits. Along the way I ran `/insights` across 79 sessions and discovered that 24.4% of my agent compute time was wasted on preventable friction. Here is what I learned, quantified, and what I changed to fix it.

---

## The Setup

OBOS is my personal operating system dashboard. React 18 + TypeScript + Vite SPA with an Express server on port 5000. All data lives as JSON files on disk -- no database, no ORM, no migrations. The server reads from `../data/` and the frontend fetches via a custom `useApi<T>(url, {refreshInterval?})` hook.

The dashboard covers everything I need to manage my professional and personal life from a single pane of glass:

- **Task Board** -- kanban with GitHub PR integration, agent session tracking, approval workflows
- **DeFi Monitor** -- Aave yield tracking, advertised vs realized APY comparison, trade signals
- **News Feed** -- RSS aggregation with keyword scoring, 3-pillar analysis (Finance, Tax & Location, Creative & AI)
- **Schengen Calculator** -- 90/180-day rolling window compliance for digital nomad travel
- **Calendar** -- ICS import with week/today views, source filtering
- **Usage & Costs** -- agent runtime tracing, token consumption, cost attribution

Plus Activity Feed, Agents gallery, Job Search, Crypto Travel conferences, Memory Browser, and a Skills registry. 12 pages total, built from Feb 17-22, 2026.

The question I wanted to answer: can Claude Code agents actually build and maintain a production-quality dashboard autonomously, or does AI-assisted coding collapse under the weight of a real multi-feature project?

---

## The Architecture

Two systems work together. **OpenClaw** is the control plane -- task dispatch, agent launching, worktree isolation, human approval gates, Telegram notifications. **Claude Code** is the execution layer -- implementation, code review, self-reflection, quality gating.

### Three Agents, Three Roles

| Agent | Model | Job |
|-------|-------|-----|
| `backend-engineer` | Claude Opus | Implementation -- writes code, runs builds, commits |
| `reviewer` | Claude Sonnet | Quality gate -- reviews diffs, returns APPROVED or REVISION_NEEDED |
| `ai-accelerator` | Claude Opus | Research -- deep product/market analysis with evidence-backed proposals |

### The Pipeline

The standard build pipeline looks like this:

1. OpenClaw writes a spec and posts it to `tasks/board.json`
2. OpenClaw launches `claude -p` in a TMUX session with worktree isolation
3. `backend-engineer` reads the spec and implements
4. `reviewer` validates -- 5-iteration loop maximum before escalation
5. `/judge` runs adversarial evaluation (more on this below)
6. `/ship` runs the final pre-push gate: type check, build, Playwright tests, selective commit, push
7. Claude Code's Stop hook fires a webhook to the OpenClaw gateway, which wakes NEXUS to process the result

Each agent operates in its own Git worktree. No merge conflicts between parallel sessions. Cleanup happens automatically after merge.

### `CLAUDE.md` as Anti-Regression Core

Every rule in my `CLAUDE.md` exists because something went wrong. It is not a style guide. It is a scar tissue document.

The file opens with six "Known Model Failure Modes" -- behaviors I observed repeatedly and encoded as permanent guardrails:

1. **Hallucinated APIs** -- Claude will confidently call functions that do not exist
2. **Post-compaction amnesia** -- after auto-compaction at ~83% context, plans and decisions vanish
3. **Overconfident validation** -- claiming success without running tests
4. **Silent drift** -- gradually ignoring instructions as context fills up
5. **Defensive code from phantom bugs** -- adding unnecessary try-catch and `any` types when confused
6. **Verbosity = uncertainty** -- writing more code than needed signals the model does not understand the task

This file survives context compaction because Claude Code loads it at session start. Conversation history does not survive. This single fact -- that `CLAUDE.md` persists but conversation does not -- is the most important architectural insight of the entire project.

### The Skills System

Four custom skills form a feedback loop:

- **`/reflect`** -- self-reflection after every feature. Produces test skeletons, Living Rules, memory corrections.
- **`/refactor`** -- deterministic cleanup using `knip` (dead code) and `jscpd` (duplication). Requires explicit approval before code changes.
- **`/judge`** -- adversarial quality evaluation. Not collaborative -- it only identifies problems, never fixes them.
- **`/ship`** -- final pre-push gate. Type check, build, Playwright tests, commit, push, reflect, GitHub comment.

The design philosophy: **deterministic tools for detection, LLM for judgment.** `knip` finds dead code. `jscpd` finds duplication. Those are exact algorithmic problems. The LLM decides what to do about the findings. Do not use AI for problems that have exact solutions.

---

## The Skills Feedback Loop

`/reflect` is the engine that makes the system learn from its own failures. After every feature, it runs a 20-question self-audit and produces three artifacts:

### 1. Test Skeletons

Concrete Playwright test stubs for the feature just built. Not aspirational test plans -- actual `test("description", async ({ page }) => { ... })` blocks ready to flesh out.

### 2. Living Rules

One-line rules with date, context, and guard. They capture HOW things broke, not just WHAT broke. Here is a real example from the Schengen calculator:

```
[2026-02-22] [schengen] RULE: Use noon-anchored dates (`new Date(str + "T12:00:00")`)
for all YYYY-MM-DD parsing.
WHY: Bare `new Date("YYYY-MM-DD")` -> UTC midnight -> wrong day in UTC-negative timezones.
Off-by-one in Schengen calc.
GUARD: "POST /api/schengen/trips creates trip with correct totalDays" in schengen.spec.ts
```

This rule exists because `new Date("2026-03-15")` parses as UTC midnight, which in a UTC-5 timezone becomes March 14 local time. The Schengen calculator was counting one fewer day than it should. Three iterations to diagnose. Appending `T12:00:00` anchors the parse to noon, making it timezone-safe. The guard ensures a test catches any regression.

Another example:

```
[2026-02-22] [playwright] RULE: Use `{ exact: true }` with `getByText()` when label
text may substring-match elsewhere.
WHY: `getByText('DAYS REMAINING')` matched stat card AND subtitle, causing strict mode failure.
3 iterations to fix.
```

### 3. Memory Corrections

Updates to the project's architecture knowledge in `MEMORY.md`. Endpoint signatures, file locations, data schemas -- anything that changed during implementation gets recorded so future sessions start with accurate context.

### The Compounding Effect

Each `/reflect` session adds approximately 3 rules. After 5 days, the system has accumulated rules covering date parsing, error handling, Playwright locator specificity, API validation, and calculation composition. Every category of failure the system has encountered is now guarded.

The critical insight: **rules in `CLAUDE.md` survive context compaction. Rules in conversation do not.** A clever prompt that says "always check for timezone issues" will be forgotten after compaction. A Living Rule in `CLAUDE.md` with a specific date, context, and guard persists indefinitely. This is why the persistent anti-regression file matters more than clever prompting.

---

## What Went Wrong -- Quantified

I ran `/insights` across 79 Claude Code sessions. 56 had usable data. 54 total hours of agent compute time over 5 days.

**24.4% friction rate** -- approximately 13.2 hours wasted on preventable issues.

| Friction Category | Sessions Hit | Est. Extra Time/Session | Total Waste | % of 54h |
|---|---|---|---|---|
| Buggy first-pass code | 22 | ~15 min | ~5.5h | 10.2% |
| Wrong architectural approach | 14 | ~30 min | ~7.0h | 13.0% |
| Git/CI infrastructure failures | 4+ | ~10 min | ~0.7h | 1.3% |
| **Total** | | | **~13.2h** | **24.4%** |

### Buggy First-Pass Code (22 sessions, ~5.5 hours)

The root cause was not that agents write bad code. They write reasonable first drafts. The problem is that **verification happened too late** -- after all implementation was "complete," not incrementally after each edit.

A typical failure pattern: agent writes 200 lines across 3 files, then runs `tsc --noEmit`, discovers 8 type errors that cascade across imports, and spends 15+ minutes unwinding. If type checking had run after each file, the errors would have been caught and fixed in context, one at a time.

I considered a `postToolUse` hook that would run `tsc --noEmit` automatically after every file edit. Benchmarked it at **5.8 seconds** on our 11,500-line codebase. Too slow -- the threshold for viability was 3 seconds. Every edit would feel like molasses. It would need `--incremental` mode to be practical. Deferred, not abandoned.

Secondary finding: `knip` flagged 12 unused type exports. Agents export types "just in case" even when nothing imports them. Low-cost waste, but it accumulates.

### Wrong Architectural Approach (14 sessions, ~7 hours)

This was the biggest time sink. Without a planning time-box, agents research indefinitely because research feels productive. It is not.

One session spent **65 minutes** researching notification pipeline architecture without writing a single line of code. The agent read documentation, compared approaches, analyzed tradeoffs -- all useful activities in moderation. But at minute 65 with zero lines written, the actual constraints of the codebase had not been tested against reality.

The NEXUS notification pipeline was the worst example. The initial approach (a `POST /api/cron/wake` endpoint) was architecturally wrong. It required a complete rethink to the current `POST /hooks/wake` webhook pattern. That architectural rethink cost an entire session.

The lesson: **a 10-minute planning cap forces first implementation attempts that reveal real constraints faster than more research would.** The codebase talks back when you write code against it. Documentation does not.

### Git/CI Failures (4+ sessions, ~0.7 hours)

Inline credential helpers with shell escaping broke repeatedly. The `git push` command would use an inline `credential.helper` with special characters in the token, and the shell would mangle it. Low total waste but high annoyance -- each occurrence blocked an otherwise-complete session at the final step.

The fix was boring: document a canonical push command in `CLAUDE.md` using `doppler` for secret retrieval, and add a rule to never improvise authentication strategies.

---

## What Fixed It

### 5 New `CLAUDE.md` Rules

Each rule traces directly to a friction category:

1. **Planning time-box (10 min max)** -- addresses the 14 wrong-approach sessions. After 10 minutes of research/planning, write what you know and start coding. Refine the plan during implementation.

2. **Agent heuristic (3-file threshold)** -- if the task touches 3 or fewer files, edit directly. If 4+, use agents. The `/insights` analysis showed 205 `Task` and 204 `TaskList` invocations, suggesting heavy over-orchestration for small changes. Multi-agent teams have overhead. Save them for genuinely parallel work.

3. **Git push strategy (canonical command, never improvise auth)** -- eliminates credential helper failures. One command, always the same, documented in `CLAUDE.md`.

4. **Mobile breakpoint standard (768px)** -- `useMediaQuery("(max-width: 768px)")` everywhere. Prevents inconsistent responsive behavior across pages.

5. **Code review diff targeting** -- always diff the working tree (`git diff`) or correct branch (`git diff main...HEAD`). Reviewing stale or wrong-branch diffs wasted entire review cycles.

### Removed a Broken Reference

`CLAUDE.md` mandated running `npm run lint` in the Verify phase. The project has no lint script in `package.json`. Agents would try to run it, get a cryptic npm error, and either fail silently or -- worse -- attempt to install ESLint mid-implementation, burning 10+ minutes on dependency resolution that was never part of the task.

The fix: delete the reference. Do not mandate tools that do not exist.

### Created the `/ship` Skill

Before `/ship`, agents would commit and push without running full verification. The skill enforces a strict pipeline:

```
tsc --noEmit -> npm run build -> npx playwright test -> selective git add -> commit -> push -> /reflect -> gh issue comment
```

Any failure at any stage is a full stop. `/ship` is not a fixer -- it is purely a verification and shipping gate. If tests fail, the agent must fix the code first. If the build fails, the agent must fix the build first. No bypasses.

### What We Did NOT Do

Considered and rejected:

- **postToolUse hook for `tsc --noEmit`** -- benchmarked at 5.8s. Needs `--incremental` mode (which requires a `.tsbuildinfo` file between invocations) to drop below the 3s threshold. Deferred to a future session.
- **Autonomous self-healing loop** -- where `/insights` findings automatically update `CLAUDE.md`. Premature. The rules need human judgment about which patterns are genuine and which are noise. Automated rule injection risks polluting the anti-regression file.
- **Rewriting Playwright tests** -- `/refactor` already identified 23 anti-patterns (7 `networkidle` waits, 8 `waitForTimeout` calls, 4 `textContent("body")` assertions). But rewriting tests is a separate workstream, not an insights-driven fix.
- **ESLint + Prettier setup** -- project setup decision, not an insights finding. Separate task.

---

## The OpenClaw + Claude Code Synergy

Two systems, complementary roles. Neither tries to do the other's job.

**OpenClaw handles**:
- Task dispatch -- writing specs to `tasks/board.json`
- Agent launching -- TMUX sessions with Git worktree isolation
- Human approval -- dashboard UI with Approve/Reject buttons
- Telegram notifications -- NEXUS posts to OB when tasks need attention
- Worktree cleanup -- automatic after merge

**Claude Code handles**:
- Implementation -- writing code, running builds
- Code review -- `reviewer` agent validates diffs
- Self-reflection -- `/reflect` captures learnings
- Quality gating -- `/ship` enforces pre-push verification

### Where Integration Works Well

The spec-to-implementation pipeline is smooth. OpenClaw writes a spec, launches an agent, the agent implements, the Stop hook calls back to the gateway, and NEXUS wakes up to process the result. End-to-end latency for a simple feature: roughly 11 seconds for the notification round-trip, plus implementation time.

Worktree isolation is critical. Two agents can work on different features simultaneously without merge conflicts. Each gets its own branch, its own working directory, its own `CLAUDE.md` context.

### Where Integration Breaks Down

Three gaps identified:

1. **No automatic `/insights` -> `CLAUDE.md` feedback loop.** After running `/insights`, the new rules must be manually added to `CLAUDE.md`. OpenClaw could trigger the analysis and Claude Code could implement the updates, but the judgment about which rules to add still requires human review.

2. **No test result visibility in OpenClaw.** When Claude Code finishes a session, OpenClaw knows the session is done (via the Stop hook). But it does not know whether tests passed. It cannot distinguish "feature complete, all tests green" from "feature pushed with failing tests." This gap means the approval gate is blind to test status.

3. **No per-issue cost attribution.** I can see total token consumption across all sessions, but I cannot tell which GitHub issues consumed the most tokens. A 3-hour debugging session and a 10-minute fix look the same in the cost dashboard. Better attribution would let me identify which types of tasks are most expensive and adjust specifications accordingly.

### The Key Architectural Insight

Separation of concerns. OpenClaw does not try to understand code. Claude Code does not try to manage task queues. They communicate through well-defined interfaces: `board.json` for task state, webhook callbacks for lifecycle events, Git branches for code delivery.

This separation makes both systems simpler and more reliable than a monolithic system that tries to do everything. When something breaks, it is immediately clear which system owns the problem.

---

## Concrete Takeaways

Seven actionable lessons for anyone using Claude Code at scale.

### 1. `CLAUDE.md` Is Your Most Important File

It survives context compaction. Conversation history does not. Every hard-won rule, every failure mode, every architectural decision should live in `CLAUDE.md`, not in your prompts or your memory.

My `CLAUDE.md` is 143 lines. It covers principles, known failure modes, mandatory workflow, agent configuration, file hygiene, git strategy, frontend conventions, testing conventions, and Living Rules. Every line is there because something went wrong without it.

### 2. Quantify Your Friction

"Things feel slow" is not actionable. "22 of 56 sessions had buggy first-pass code, costing approximately 5.5 hours" tells you exactly where to invest. Run `/insights` (or build your own session analysis). Categorize the waste. Measure it. Then fix the biggest category first.

My 24.4% friction rate means that for every 4 hours of agent compute, roughly 1 hour was wasted. That number gives me a clear target: get it below 10% within 30 days.

### 3. Time-Box Planning

Agents will research forever if you let them. 10 minutes of planning, then start coding. The implementation reveals constraints that research never will.

One session spent 65 minutes planning without writing a line. The actual problem was discovered in the first 5 minutes of implementation. That is 60 minutes of pure waste disguised as diligence.

### 4. Do Not Over-Orchestrate

Multi-agent teams have overhead. If the task touches 3 or fewer files, just do it directly. Save agents for genuinely parallel work or multi-file changes that benefit from separate builder and reviewer roles.

My `/insights` analysis showed 205 `Task` calls and 204 `TaskList` calls -- strong evidence that small tasks were being over-orchestrated with unnecessary agent infrastructure.

### 5. Build Skills That Compound

`/reflect` produces Living Rules. Living Rules go into `CLAUDE.md`. `CLAUDE.md` loads at the start of every session. Each session makes the next one slightly less error-prone.

This compounding is the real value of the skills system. Not individual skill invocations, but the accumulated learning. After 5 days, the system has rules covering date parsing, error handling, Playwright locators, API validation, and calculation composition. Each rule prevents a specific class of failure from recurring.

### 6. Deterministic Tools for Detection, LLM for Judgment

`knip` finds dead code. `jscpd` finds duplication. These are exact algorithmic problems with deterministic answers. The LLM decides what to do about the findings -- which dead code to remove, which duplication is acceptable, which refactors are worth the risk.

Do not use AI for problems that have exact solutions. Use AI for problems that require judgment.

### 7. Separate Control Plane from Execution

OpenClaw dispatches tasks and manages approval workflows. Claude Code implements and self-validates. Neither system tries to do the other's job.

This separation makes both systems simpler and more reliable. OpenClaw does not need to understand TypeScript. Claude Code does not need to manage Telegram notifications. They communicate through `board.json`, webhook callbacks, and Git branches. Clean interfaces between systems are more maintainable than monolithic systems that do everything.

---

## What's Next

Four improvements on the roadmap:

- **Investigate `tsc --incremental`** for viable `postToolUse` hooks. The current 5.8s benchmark needs to drop below 3s. Incremental type checking with a persistent `.tsbuildinfo` file should get us there.

- **Build automatic `/insights` -> `CLAUDE.md` feedback loop** via OpenClaw. The analysis runs automatically, proposes rule additions, and a human reviews before they are committed. Closes the biggest integration gap.

- **Add test result reporting** from Claude Code sessions to the OpenClaw dashboard. The approval gate should show whether tests passed, not just whether the session completed.

- **Explore per-issue cost attribution** for better resource allocation. Know which types of tasks consume the most tokens so specifications can be adjusted accordingly.

The goal: reduce the 24.4% friction rate to below 10% over the next 30 days. That is roughly 8 hours of agent compute time recovered per week -- enough to build another dashboard feature every cycle.

---

*Built with Claude Code. Analyzed with Claude Code. Written about by Claude Code.*
