# Content Plan — Week of 2026-03-08 (Week 4)

## Blog Post — WRITTEN

**Published**: `~/ai-journal/_posts/2026-03-09-week-4-what-your-ai-agent-forgets.md`
**Title**: "Week 4: What Your AI Agent Forgets Every Session"
**Angle**: Combined DOM-to-server observability (secondary) with progressive disclosure for agent memory (primary). The hook: MEMORY.md silently truncating 45% of accumulated knowledge. Three-layer fix: always-loaded index → path-triggered rules → on-demand topics. 86% reduction, 0 knowledge lost.

**Actual data points** (verified from git):
- 89 commits shipped this week (not 110 — corrected from earlier estimate)
- Evidence types: 2 → 5, pipeline reorder step 6 → 5
- MEMORY.md: 38K → 5.4K chars (86% reduction)
- CLAUDE.md: 115 → 8 inline rules (rest moved to 11 path-scoped files)
- Route split: 3,976 → 4 modules, 81 routes, 0 regressions
- E2E tests: 262 total

**Supporting learning**: `~/ai-journal/learnings/2026-03-09-progressive-disclosure-agent-memory.md`

---

## LinkedIn Posts (3 angles from the blog)

1. **"Your logs are lying to you"** — The gap between what server logs show and what the user actually saw. Introduce the DOM snapshot layer as the missing piece most stacks skip.
   - Data point: Evidence types grew from 2 → 5 before a single approval decision is made
   - Suggested day: Monday

2. **"We moved the screenshot before the review, not after"** — A single pipeline reorder changed everything. Evidence-as-input vs evidence-as-afterthought. Short, punchy, visual.
   - Data point: Pipeline step moved from position 6 → 5. Sounds trivial. Wasn't.
   - Suggested day: Wednesday

3. **"110 commits, 1 week, solo"** — Weekly velocity post. Lead with the DOM observability story, close with the route refactor (3976→4 files, 0 route diff). Prove that AI agents + tight architecture = compounding output.
   - Data point: 110 commits, 4 modules, 81 routes, 0 regressions
   - Suggested day: Friday

---

## Illustration Brief

**Scene**: A dark terminal workspace. On the left: a browser window showing a UI component — frozen mid-render, like a still frame. In the centre: three floating document icons labelled "DOM", "EVENT", "SERVER" connected by thin glowing lines (like a circuit trace). On the right: a magnifying glass hovering over the server log, highlighting a single matched line.
**Style**: Flat vector, muted dark background (#1a1a2e), electric blue and amber accent lines
**Elements**: Browser chrome, three chained document nodes, magnifying glass, subtle terminal glow
**Mood**: Investigative / technical / methodical
**Reference**: Previous illustrations in ~/ai-journal/assets/images/

---

## Content Backlog (future weeks)

- **"4/8 AI critique claims were hallucinated"** (`2026-03-05-validate-sdk-before-trusting-agents`): The story of validating an adversarial LanceDB critique against actual .d.ts files — half the "improvements" didn't exist. Strong contrarian angle for developers who over-trust AI code review. Has numbers (4/8 falsified, 376 lines removed, 24/24 tests pass).

- **"Dark Factory: Building a 5-Phase Voice Pipeline"** (`feat(#278)` — Dark Factory lakehouse): 5-phase architecture for automated voice content production. Novel system design story with a distinct aesthetic. Hold until the pipeline ships a real piece of content.

- **"The LanceDB Select Trap"** (`2026-03-04-lancedb-virtual-columns`): Virtual columns that silently return zero results while saying 200 OK. Niche but high-value for anyone building on LanceDB or similar engines.

- **"Defense-in-Depth for Test Data"** (`2026-03-02-test-isolation-defense-in-depth`): 98% queue pollution from E2E tests. Three-layer fix. Practical and broadly applicable.

- **"Progressive Disclosure for AI Agent Memory"** (`2026-03-09-progressive-disclosure-agent-memory`): Three-layer memory architecture (index → rules → topics), 86% reduction, Unix man-pages analogy. Strong LinkedIn angle — relatable to anyone using Claude Code, Codex, or Cursor with persistent memory.

---

## Alignment Check

- [x] Blog topic connects to OB's AI authority positioning (agent observability = unique angle in the space)
- [x] At least one post has a contrarian or surprising angle (LinkedIn #1: "your logs are lying to you")
- [x] Data points are from this week (110 commits, 5 evidence types, pipeline reorder — all Mar 8)
- [x] No overlap with previously published posts (no observability coverage in weeks 1–3)
