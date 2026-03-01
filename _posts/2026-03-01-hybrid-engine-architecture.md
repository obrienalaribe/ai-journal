---
layout: post
title: "The Hybrid Engine: Routing AI Agents by Problem Shape"
date: 2026-03-01
author: OB
---

Last week I [ran Codex and Claude Code against the same 10 bugs](/ai-journal/2026/03/01/codex-vs-claude-code.html). Codex won on speed. Claude Code won on safety. The obvious question: what if you didn't have to choose?

This week I built the system that answers that question.

![Hybrid Engine Router — issues flow through a spec quality gate to either the Codex fast path or Claude Code deep path](/ai-journal/assets/images/hybrid-engine-router.png)

## The Core Idea

Route every GitHub issue to the right engine based on **problem shape**, not model preference:

- **Codex** (`engine:codex`): Tool-shaped problems. Autonomous, single-shot. Well-specified bugs with clear acceptance criteria.
- **Claude Code** (`engine:claude`): Colleague-shaped problems. Iterative, multi-step. Enhancements, architectural work, ambiguous scope.

The insight that made this click: **Codex's effectiveness is bounded by spec quality, not model capability.** If you write a vague spec, Codex will faithfully implement the wrong thing. If you write a CNC program, Codex will execute it exactly.

## The Architecture

The system has five layers, built in five phases over one session.

### Layer 1: Engine Routing

The agent-task-runner cron picks up GitHub issues every 30 minutes. Before spinning up an agent, it now asks: which engine?

```
Issue arrives
    |
    v
Explicit label? ── engine:codex ──> Codex path
    |                engine:claude -> Claude Code path
    v
Previous Codex attempt failed 2x? ──> Claude Code
    |
    v
Label = bug? ──> Codex
Label = enhancement? ──> Claude Code
    |
    v
Default ──> Codex
```

Explicit labels always win. After that, it's heuristics. Bugs default to Codex because they're usually well-scoped. Enhancements default to Claude Code because they need creative judgment.

### Layer 2: Sandboxed Implementation + External Validation

This is the key architectural decision. Codex runs in `--full-auto` mode (sandboxed — can't bind ports or run E2E tests). Then validation runs *externally* after Codex exits:

```bash
# Codex implements (sandboxed)
cat $PROMPT | codex exec --full-auto --add-dir $WORKTREE

# External validation (not sandboxed)
cd $WORKTREE && tsc --noEmit && npm run build && npx playwright test
```

Codex never needs to self-validate. The validation is deterministic and model-independent. This matches the existing Claude Code pattern where `/ship` is the external gate — we just moved the gate outside the agent.

If validation fails, the issue gets re-labeled `engine:claude` and the next cron cycle picks it up with the full pipeline.

### Layer 3: The Spec Quality Gate

![Spec Quality Gate — six checkpoints that determine if a spec is CNC-grade or conversation-grade](/ai-journal/assets/images/spec-quality-gate.png)

This is where the real leverage lives. Before any issue gets `engine:codex`, it must pass six checks:

1. **Current behavior stated** — what does the system do now?
2. **Expected behavior stated** — what should it do instead?
3. **Root cause identified** — file path and function name
4. **Fix boundary scoped** — which files will change?
5. **Acceptance criteria are test-verifiable** — not "should feel better"
6. **No design decisions remain** — no "choose the best approach"

All 6 pass? `engine:codex`. Any fail? `engine:claude`.

Here's a spec that passes all six:

> **Bug: pipeline-phases.json not seeded on startup**
>
> Current: `GET /api/content/pipeline/status` returns `{ phases: [] }` on fresh install
>
> Expected: Returns 7 phases matching `config/content/pipeline-phases.json`
>
> Root cause: `seedContentFiles()` only seeds `SEED_FILES` map, not config JSON
>
> Fix: `routes/content.js` → `seedContentFiles()` function
>
> Acceptance: `npx playwright test tests/e2e/content.spec.ts` passes, `tsc --noEmit` clean

That's a CNC program. Codex executes it in 8 minutes.

Here's one that fails:

> **Enhancement: Rework Intelligence column**
>
> The intelligence tab needs reworking. Ideas shouldn't have platform badges. Planning column should handle channel assignment. Make it feel more like a triage inbox.

"Feel more like" is colleague territory. You need back-and-forth to discover what "triage inbox" means concretely. That's Claude Code's job.

### Layer 4: The Feedback Loop

Every Codex run gets logged to `engine-metrics.jsonl`:

```json
{
  "issue": 197,
  "engine": "codex",
  "attempt": 1,
  "tokens": 121000,
  "timeMs": 480000,
  "result": "pass",
  "testsFixed": 10,
  "reviewerFlags": ["unnecessary_host_change"]
}
```

Weekly analysis answers three questions:
- What percentage of Codex runs succeed first-pass?
- Which spec gaps caused failures?
- Which issue types consistently need Claude Code?

Failed Codex runs are **spec quality signals**. When Codex fails, I ask: could the spec have prevented this? If yes, the spec template gets updated. If no, that issue class isn't Codex-shaped.

Over time this builds a corpus of "specs that work for Codex" vs "specs that need Claude Code." The routing heuristics get sharper with every failure.

### Layer 5: Reflection Decoupling

The existing Claude Code pipeline runs `/reflect` after every issue — a 20-question self-audit that generates test skeletons, updates Living Rules, and corrects auto-memory. That's valuable but expensive per-issue.

For Codex runs, reflection becomes **periodic instead of tactical**:

- Weekly `reflect-sweep` cron (Sunday 3am Paris time)
- Claude Code reviews all commits from the past week
- Generates Living Rules updates from patterns, not individual fixes
- Files issues for anything actionable

Reflection shifts from "what did I learn from this commit" to "what patterns emerged this week." Strategic, not tactical.

## The Performance Data

From the first production run (Issue #197, 10 E2E test failures):

| Metric | Codex Path | Claude Code Path |
|--------|-----------|-----------------|
| Time to fix | 8 min | 2+ hours (unresolved) |
| Tokens | 121K | 500K+ estimated |
| Files patched | 5 | PR still in progress |
| Fix rate | 10/10 | Partial |
| Iterations | 1 | Multiple cycles |

The cost curve matters. At 121K tokens per Codex run vs 500K+ for the full pipeline, routing 60-70% of issues through the fast path cuts token spend by roughly half.

## The System Evolution

![System evolution — three phases from single-engine to hybrid architecture](/ai-journal/assets/images/system-evolution.png)

Looking at the progression:

**Phase 1** (what I had): Claude Code does everything. NEXUS reviews on heartbeats. Safe, slow, expensive.

**Phase 2** (Option C): Auto-merge removes NEXUS from the critical path. Deterministic hooks replace manual reviews. Faster, still single-engine.

**Phase 3** (hybrid): Codex handles the fast path. Claude Code handles depth. NEXUS writes specs and routes. Each component does what it's best at.

Each phase stripped a layer of overhead. Phase 3 is where the cost/speed curve bends — not because either model got better, but because problems got routed to the right tool.

## What Actually Changes vs What Stays

**Stays the same**: Worktree creation, branch naming, PR creation, auto-merge hooks, reviewer scans, dependency checking, sibling PR overlap detection.

**Changes**: Engine routing in the runner, external validation (post-Codex instead of internal /ship), periodic reflection instead of per-issue, spec quality enforcement before routing.

The infrastructure investment compounds. Everything I built for the Claude Code pipeline — the safety gates, the test hooks, the merge validation — now serves both engines. Codex gets a free verification layer it couldn't build for itself.

## The Spec Quality Insight

Here's the part that surprised me: **my job changed**.

Before the hybrid engine, my role was "file issues, let agents handle it." Now my highest-leverage activity is writing Codex-grade specs. A 5-minute investment in spec precision saves 30-60 minutes of agent fumbling.

The constraint section in specs is especially powerful. Instead of relying on CLAUDE.md (which Codex doesn't read the same way), I bake Living Rules directly into each issue:

```markdown
### Constraints
- Do NOT modify server.js host binding
- React hooks must come before early returns
- Use ?? [] for nullable array access from API data
- Route literal paths BEFORE parameterized routes
```

Those four lines encode 4 Living Rules that took hours of debugging to discover. Codex gets them for free, in-context, exactly where they're relevant.

## The Uncomfortable Takeaway

The discourse around AI coding tools is obsessed with which model is smarter. GPT-5.3 vs Claude Opus vs Gemini — benchmark wars, arena rankings, vibes-based comparisons.

None of that matters as much as **problem routing**.

A mediocre model with a perfect spec will outperform a brilliant model with a vague spec. The constraint isn't intelligence — it's the translation layer between human intent and machine execution.

The hybrid engine isn't really about Codex vs Claude Code. It's about building a system that treats spec quality as a first-class engineering concern. The engine routing is just the mechanism that makes spec quality *measurable*.

When a Codex run fails, the spec was bad. When it succeeds, the spec was sufficient. Over time, you learn exactly what "sufficient" means for your codebase.

That's the compounding part. Not better models — better specs.

---

*All five phases of the hybrid engine are live in production. Engine metrics are tracked in the OBOS dashboard. The system processed its first batch of auto-routed issues on March 1, 2026.*
