---
layout: post
title: "Week 3: Sharing Skills Across Models with Symlinks"
date: 2026-03-05
author: OB
tags: [claude-code, codex, skills, weekly, symlinks, cross-model]
excerpt: "131 commits, 58 Living Rules, and an experiment: symlinking Claude Code skills into OpenAI Codex. The results were more interesting than expected."
---

## This Week in Numbers

| Metric | Value |
|--------|-------|
| Commits | 131 |
| Features shipped | 51 |
| Bugs fixed | 53 |
| Living Rules added | 58 |
| Skills (Claude Code) | 11 |
| Skills shared to Codex | 11 (via symlinks) |
| Intelligence Engine rating | 3/10 (adversarial audit) |

This was the most productive week yet. The content delivery pipeline went from concept to fully automated posting. The Intelligence Engine got a 4-agent adversarial audit. And I did something that shouldn't work but does: shared skills between two competing AI coding agents using Unix symlinks.

## What We Built

- **Content Delivery R0-R4**: Full queue lifecycle from idea approval to LinkedIn posting with photo attachments. R0 (smoke test), R1a (queue hardening), R1b (n8n callbacks), R2 (story-powered generation with Anthropic API), R3 (photo metadata + image posting fallback), R4 (magic byte validation for uploaded photos).
- **FULLY AUTO pipeline**: One-click approval now auto-generates LinkedIn atoms and auto-queues with next-day scheduling. Fire-and-forget.
- **Intelligence Engine v2**: LanceDB-backed semantic search across 9 source types (576 embeddings). Workspace memory ingestion, video signal tracking, auto-reindex. Trending chips with semantic idea candidates.
- **Photo Story Journal**: Upload photos, attach to stories via interview wizard, preview in Story Bank.
- **Hybrid Engine**: Codex + Claude Code routing by problem shape. Spec quality gate determines which model handles each issue.
- **Adversarial Intelligence Audit**: 4 parallel reviewer agents found 3 critical bugs in the scoring pipeline. Reranker was emitting noise. Hybrid scores silently zeroed by a camelCase typo.
- **Aave DeFi stack**: Position dashboard with HF trend sparkline, liquidation alerts to Telegram, governance proposal monitor, accumulation score composite signal.

## The Experiment: Sharing Skills Across Models

Here's the setup. I have 11 custom skills for Claude Code:

```
~/.claude/skills/
  content-pipeline/   judge/        reflect/
  content-plan/       lancedb/      ship/
  frontend/           refactor/     syndicate/
  telegram-troubleshoot/            weekly-report/
```

Each skill is a `SKILL.md` file (YAML frontmatter + markdown instructions) with optional `references/` and `scripts/` directories. They encode hard-won operational knowledge: the `/reflect` skill runs a 20-question self-audit protocol. The `/ship` skill is a 4-phase quality gate. The `/judge` skill applies adversarial evaluation with "generous skepticism."

Last week I ran Codex against the same bugs as Claude Code and [documented the results](https://obrienalaribe.github.io/ai-journal/2026/03/01/codex-vs-claude-code.html). Codex was faster but lacked the institutional knowledge baked into my skills. So I asked: what if I just... gave Codex the same skills?

```bash
# The actual commands
mkdir -p ~/.codex/skills
for skill in ~/.claude/skills/*/; do
  ln -s "$skill" ~/.codex/skills/$(basename "$skill")
done
```

Eleven symlinks. Each pointing from `~/.codex/skills/<name>` to `~/.claude/skills/<name>`. Single source of truth, two consumers.

### Why It Works

Both Claude Code and Codex use the same skill format:

| Feature | Claude Code | Codex |
|---------|-------------|-------|
| Skill file | `SKILL.md` | `SKILL.md` |
| Frontmatter | YAML (`name`, `description`) | YAML (`name`, `description`) |
| Body | Markdown instructions | Markdown instructions |
| Subdirs | `references/`, `scripts/` | `references/`, `scripts/`, `agents/`, `assets/` |
| Loading | Auto-loaded into context | Auto-loaded into context |

The YAML frontmatter schema is identical. The markdown body is model-agnostic by nature --- it's just instructions. Both tools scan their `~/.{tool}/skills/` directory on startup and inject matching skills into the system prompt.

![Pencil sketch: Two boxes labeled "Claude Code" and "Codex CLI" with arrows pointing down to a shared cylinder labeled "~/.claude/skills/". Symlink arrows from a second box "~/.codex/skills/" curve back to the same cylinder. Caption: "One source of truth, two consumers."](/ai-journal/assets/images/skill-symlink-architecture.png)

### What Actually Happens

When Codex loads a symlinked skill like `/reflect`, it gets the full 20-question protocol, the Playwright test skeleton generator, the Living Rules update format. It doesn't know or care that these were written for Claude Code. The instructions are procedural: "Run these commands. Check these patterns. Write these outputs."

The skills that transferred best:

1. **`/judge`** --- Pure evaluation criteria. Works identically across models.
2. **`/lancedb`** --- Technical reference (index types, query patterns, schema). Model-agnostic.
3. **`/frontend`** --- Design rules, coding standards, component patterns. Just documentation.

The skills that transferred worst:

1. **`/ship`** --- References `gh` CLI patterns, branch naming conventions, and CLAUDE.md-specific Living Rules. Codex can follow the commands but lacks the context of why each gate exists.
2. **`/reflect`** --- The 20-question protocol references CLAUDE.md sections and Living Rule format. Codex writes the artifacts but doesn't know where to put them (it doesn't have auto-memory or `CLAUDE.md` conventions).

### The Pros

**Single source of truth.** Edit a skill in `~/.claude/skills/`, both tools see the update immediately. No sync, no copy, no drift.

**Zero-cost experimentation.** Symlinks are instant. No migration, no config file changes, no restart of the other tool.

**Institutional knowledge portability.** The hard-won patterns from 500+ hours of Claude Code sessions become available to Codex without rewriting anything. Codex gets the benefit of 58 Living Rules distilled into skill instructions.

**Model arbitrage.** Different models have different strengths. Codex is faster for single-shot fixes. Claude Code is better for multi-step orchestration. With shared skills, you route by problem shape while maintaining consistent quality standards.

### The Cons

**Context window pressure.** Codex's skill-creator docs warn: "The context window is a public good." Claude Code skills were written assuming Opus-sized context. Some skills (like `/content-pipeline` at 350 lines) may be too heavy for models with smaller windows.

**Tool-specific references.** Skills that reference `CLAUDE.md`, Living Rules, auto-memory, or Claude Code-specific hooks break when loaded by Codex. Codex has `~/.codex/memories/` (empty in my case) and `config.toml` --- different conventions.

**Codex `.system` skills conflict.** Codex ships with built-in `skill-creator` and `skill-installer` in `.system/`. These have their own conventions (like `agents/openai.yaml` for UI metadata). Symlinked Claude skills don't have these, which means they work functionally but don't appear in Codex's skill listing UI.

**No CLAUDE.md equivalent in Codex.** Claude Code auto-loads `CLAUDE.md` from the project root. Codex uses `config.toml` with trust levels and `instructions.md` (which I haven't set up). Skills that assume CLAUDE.md conventions need adaptation.

**Bidirectional editing risk.** If Codex modifies a symlinked skill file (say, during a `/reflect` session), it modifies the Claude Code original. This is a feature if you trust both tools. It's a bug if Codex rewrites a skill in a format Claude Code doesn't expect.

![Pencil sketch: A bridge labeled "Symlinks" connecting two islands. Left island: Claude Code with a tall tower of stacked blocks (CLAUDE.md, hooks, auto-memory, Living Rules). Right island: Codex with a simpler structure (config.toml, memories/, trust levels). The bridge supports foot traffic but not the tower. Caption: "Skills cross the bridge. Infrastructure doesn't."](/ai-journal/assets/images/skill-bridge-limits.png)

### Key Insight: Skills Are the Portable Layer

The experiment revealed something about AI coding tool architecture:

```
┌─────────────────────────────────────────────┐
│              INFRASTRUCTURE                  │  <-- NOT portable
│  CLAUDE.md, hooks, auto-memory, Living Rules │
│  config.toml, memories/, trust levels        │
├─────────────────────────────────────────────┤
│              SKILLS                          │  <-- PORTABLE
│  SKILL.md + references/ + scripts/           │
│  Procedural knowledge, evaluation criteria   │
├─────────────────────────────────────────────┤
│              MODEL                           │  <-- Different
│  Opus, Sonnet, o3, GPT-4.1                   │
└─────────────────────────────────────────────┘
```

Skills sit in the middle layer. They're more structured than raw prompts but less coupled than infrastructure. A well-written skill is essentially a runbook --- and runbooks don't care which engineer reads them.

The infrastructure layer (hooks, auto-memory, Living Rules, CLAUDE.md) is where the real lock-in lives. That's the part that makes Claude Code feel like a colleague rather than a tool. Codex doesn't have this layer yet (its `memories/` directory is empty, its `config.toml` is 3 lines). But it will. And when it does, the skill symlink pattern means you've already been building for both.

![Pencil sketch: Three horizontal layers like a sandwich. Top slice labeled "Infrastructure" with a padlock icon. Middle filling labeled "Skills" with bidirectional arrows. Bottom slice labeled "Model" with different brand logos. An arrow from the middle layer goes both left and right. Caption: "The portable middle."](/ai-journal/assets/images/portable-middle-layer.png)

## What Went Wrong

### Intelligence Engine: 3 Critical Bugs

The adversarial audit was the week's biggest reality check. Four parallel reviewer agents discovered:

1. **`_relevanceScore` camelCase typo** (`routes/intelligence.js:216`): LanceDB RRF returns `_relevance_score` (snake_case). One character wrong = 50% of the corpus scored zero. OB's own blog posts and learnings were invisible in every search.

2. **Cross-encoder tokenizer format** (`routes/intelligence.js:168`): `tokenizer([query, doc])` tokenizes them as separate sequences. The model scores each independently. Correct: `tokenizer(query, { text_pair: doc })`. Wrong format produced 0.21-point spread. Correct format: 16.3. The reranker was emitting noise.

3. **FTS/hybrid 182x scale mismatch**: BM25 scores (0-10) vs RRF scores (0-0.03). News articles always dominated. A generic AI payment rails article outranked OB's own Claude Code blog post for "AI agents coding workflow."

Combined effect: 0 out of 10 realistic content queries returned useful results.

### Hybrid Engine Scope Creep

The Codex integration experiment spawned a 5-phase hybrid engine build that consumed 2 days. The `pick-engine.mjs` router, the spec quality gate, the NEXUS-free merge pipeline --- each individually justified, but the combined effort was disproportionate to the number of Codex-routed issues (roughly 4 per week).

### 58 Living Rules

Fifty-eight new Living Rules in one week means fifty-eight failure modes encountered. Some highlights:

- E2E tests that POST data must clean up in `afterAll` --- n8n picked up test entries and tried to post them to LinkedIn (#209)
- Production endpoints must filter test-prefixed items server-side --- defense-in-depth when cleanup fails
- `git stash drop` without verifying `pop` succeeded = lost all work and had to reimplement from scratch

## What Fixed It

The Living Rules system is now self-healing. Each rule follows the format:
```
[DATE] [context] RULE: what to do
WHY: what went wrong (specific incident)
GUARD: test or grep that detects regression
```

CLAUDE.md auto-loads into every session, so every agent inherits every rule. The `/reflect` skill forces rule extraction after each feature. The `/intelligence` audit skill spawns adversarial reviewers to find bugs that the builder missed.

The specific intelligence bugs are surgical fixes (3 one-line changes). Filed as [#261](https://github.com/obrienalaribe/obos/issues/261).

## Lessons Learned

1. **Symlinks are the simplest cross-model skill sharing.** No SDK, no migration tool, no config sync daemon. `ln -s` works because both tools converged on the same format independently. Check if it still works before building something complex.

2. **Skills are portable; infrastructure isn't.** The skill file format is the universal layer. CLAUDE.md, hooks, auto-memory, trust levels --- that's where the differentiation (and lock-in) lives. Write skills that reference procedural commands, not tool-specific features.

3. **Adversarial audits find bugs that tests don't.** 23 E2E tests passed with the scoring pipeline completely broken. Shape assertions (`response has results array`) don't catch ranking quality. The 4-agent audit pattern --- search quality, LanceDB practices, content strategy, scoring pipeline --- found 3 P0 bugs in 12 minutes.

4. **Living Rules accumulate faster than you expect.** 58 rules in one week. At this rate, CLAUDE.md will hit 200 rules by April. Need consolidation before adding more. The value curve flattens --- rule #5 prevents more failures than rule #55.

5. **The 10% Rule applies to infrastructure.** The hybrid engine was technically elegant but solved a 4-issues-per-week problem with a 5-layer architecture. A simpler approach: just label issues manually. Match complexity to frequency.

## Living Rules Added This Week

58 rules added (too many to list individually). Key themes:

**Content delivery** (12 rules): Queue lifecycle, rate caps, test isolation, serial test ordering, cleanup patterns.

**LanceDB/Intelligence** (8 rules): Schema column selection, virtual column naming, mergeInsert patterns, cross-encoder tokenizer format, SEARCH_SELECT deprecation.

**Playwright testing** (9 rules): Serial mode for shared state, stale server detection, CSS text-transform vs DOM text, fixture creation in test setup.

**Server patterns** (11 rules): Route ordering (literal before parameterized), factory injection, seed-on-first-read, config fallback chains, Express route shadowing.

**UI/Frontend** (7 rules): Nested modal z-index, phase-aware card rendering, useApi conventions, ScoreCircle size variants.

**Process** (5 rules): git stash safety, evidence capture, n8n container protection, MEMORY.md location disambiguation.

Full list: [CLAUDE.md Living Rules section](https://github.com/obrienalaribe/obos/blob/main/CLAUDE.md).

## What's Next

- **Fix Intelligence Engine** (#261): Apply the 3 one-line bug fixes. Verify reranker spread >10. Verify OB's blog posts surface in top 3 for relevant queries.
- **Content quality improvements** (#262): Source-type weighting (stories 3x, news 0.3x), workspace memory noise filtering, all 3 pillars indexed.
- **Consolidate Living Rules**: 58 new rules need clustering. Many are variants of the same pattern ("verify before assuming"). Target: consolidate to <40 by grouping.
- **First real content publish**: The auto pipeline is built. Next week: approve an idea, let the system generate + queue + post to LinkedIn. Measure: did it actually sound like OB?

---
*Week 3 of building with autonomous AI agents. 131 commits. 58 rules learned. One symlink experiment that suggests the future of AI tooling is more interoperable than the vendors want you to think.*

*Generated from /intelligence audit + git history. Reviewed and published by OB.*
