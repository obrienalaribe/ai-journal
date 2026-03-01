---
layout: post
title: "Building a News Aggregation Pipeline That Actually Works"
date: 2026-02-24
author: OB
tags: [claude-code, news-pipeline, rss, content-filtering, ai-analysis]
excerpt: "17 RSS feeds. 5000+ articles per week. 45 that matter. How bucketed keyword scoring and LLM analysis replaced the firehose with a focused intelligence brief."
---

I was drowning in tabs. CoinDesk, Decrypt, TechCrunch, The Block — every morning started with 30 minutes of scanning headlines across a dozen sites, trying to figure out what mattered for my three areas of focus: crypto finance, tax/location strategy, and AI tooling.

So I built a news pipeline inside OBOS. Not an RSS reader. A filtering machine that takes 5000+ articles per week from 17 sources and reduces them to 45 — the ones that actually deserve attention. Then an LLM summarizes them into a three-pillar intelligence brief.

Here's how it works, what broke along the way, and the architectural decisions that made it reliable.

---

## The Architecture

The pipeline has four stages:

```
17 RSS Feeds → 30-Day Hard Cutoff → Bucketed Keyword Filter → LLM Analysis
     ↓              ↓                       ↓                      ↓
  ~5000/wk      Drop stale           45 articles max         3 pillar briefs
                 backfill            (15 per pillar)         with action items
```

Each stage is deliberately simple. No ML classifiers. No embedding similarity. No vector databases. Just RSS parsing, string matching, time bucketing, and one Anthropic API call at the end.

### Stage 1: RSS Ingestion

17 sources across four categories:

| Category | Sources | Examples |
|----------|---------|----------|
| Crypto (6) | CoinDesk, Decrypt, The Block, CoinTelegraph, DL News, CryptoPotato | Market moves, regulation, DeFi |
| AI/Security (5) | TechCrunch AI, MIT Tech Review, The Verge AI, n8n Blog, The Hacker News | Model releases, tooling, security |
| Tech (3) | HackerNews Best, Ars Technica, The New Stack | Engineering practices, infra |
| Nomad (3) | IMI Daily, Euronews, VisaVerge | Visa policy, residency, tax law |

Every 30 minutes, the server polls all enabled feeds using `rss-parser`. Articles are deduplicated by SHA-256 hash of `sourceId:link` and by normalized title. New articles merge into a daily JSON file.

The dedup was necessary because RSS feeds backfill — when you first add a source, it dumps 50+ articles from its archive. Without the hash + title dedup, you get duplicates within hours.

### Stage 2: The 30-Day Cutoff

A hard time filter drops anything older than 30 days. This sounds obvious but it solved a real problem: RSS feeds from some sources include articles from months ago in their feed. Without the cutoff, stale content would crowd out fresh articles in the keyword filter.

### Stage 3: Bucketed Keyword Scoring

This is where it gets interesting. The naive approach would be: score every article by keyword matches, sort by score, take the top N. We tried that first. It failed.

**Why weighted scoring doesn't work:**

A 3-day-old article about "Bitcoin ETF regulation" (score: 8, two priority terms) would rank higher than a 2-hour-old article about "Bitcoin crash below $65K" (score: 3, one keyword match). The old article has more keyword hits, but the fresh article is what you actually need to read today.

Weighted scoring mixes two axes — relevance and freshness — into one number. When you flatten two dimensions into one, you lose the ability to reason about either.

**The fix: freshness-first bucketing.**

```
Bucket 1: <24 hours    → sort by keyword score within bucket
Bucket 2: <48 hours    → sort by keyword score
Bucket 3: <7 days      → sort by keyword score
Bucket 4: 7-30 days    → sort by keyword score
```

Fill slots from Bucket 1 first, then Bucket 2, and so on. Within each bucket, higher keyword scores win. This guarantees that fresh articles always beat older ones, regardless of score — while still using relevance to pick between articles of similar age.

The keyword model has two tiers:
- **Priority terms** (+5 points): "CLARITY Act", "stablecoin regulation", "ETIAS", "Claude Code" — terms where any match means the article is almost certainly relevant
- **Standard keywords** (+1 point): broader terms like "bitcoin", "yield", "visa" — useful for scoring but not sufficient alone

Combined with per-pillar caps (`maxPerPillar: 15`), this prevents any single pillar from dominating the output. The finance pillar could easily consume all 45 slots on a volatile market day. Caps keep the other pillars visible.

### Stage 4: LLM Analysis

The filtered 45 articles go to Anthropic's API (Sonnet) as three parallel requests — one per pillar. Each pillar gets a tailored extraction prompt:

- **Finance**: "OB's goals: cbBTC lending strategy and USDC accumulation"
- **Tax & Location**: "OB's goals: zero capital gains tax, digital nomad visa, $1300/mo beach city"
- **Creative AI**: "OB's goals: AI brand authority, YouTube growth, AI strategist positioning"

The LLM returns structured JSON: a top signal (one headline sentence), five bullet-point summary, action items, and top articles. Three parallel calls complete in 5-8 seconds.

If no API key is available, the system falls back to NEXUS (my agent orchestration gateway) to trigger analysis asynchronously.

---

## What Broke (And How I Fixed It)

### Dead RSS Sources

The gap analysis revealed 2 of our original 10 sources were completely broken:
- **InfoQ**: returned HTTP 406 (Not Acceptable)
- **Nomad Capitalist**: returned malformed XML

Neither failure was visible in the dashboard because errors were silently caught. After fixing the error surfacing, I added a rule: always `curl -sI` every RSS URL before adding it to the sources list. Verify 2xx status and XML content-type.

### Arc Publishing RSS Discovery

DL News and CoinDesk both run on Arc Publishing. Their `/rss/` path returns an HTML page, not an RSS feed. The actual feed URL follows a different pattern:

```
{domain}/arc/outboundfeeds/rss/?outputType=xml
```

I discovered this by inspecting CoinDesk's working URL (which we already had) and noticing the pattern. Applied it to DL News, and it worked immediately. Added a Living Rule so this knowledge persists:

> [2026-02-23] [news-feed] Arc Publishing sites use `/arc/outboundfeeds/rss/?outputType=xml` — never `{domain}/rss/`.

### The Tax Pillar Was Completely Empty

The original 10 sources were all crypto or tech publications. Zero nomad/visa/expat content. The tax_location pillar produced 0 articles — not because the filter was wrong, but because there was nothing to filter.

The fix was source expansion: IMI Daily (investment migration), VisaVerge (visa news), Euronews (European policy), and Travel Off Path (nomad destinations). After adding these four sources, the tax pillar went from 0 to 11 articles per day.

### Cache-Control Defense-in-Depth

Users were seeing stale news summaries because `index.html` was being cached by the browser. I set `Cache-Control: no-cache, no-store, must-revalidate` on the Express static file handler — but the SPA fallback route (which handles all non-file paths) didn't have the same header.

The fix: set cache headers in both places. Added a Living Rule:

> Always set Cache-Control on index.html at 2+ points: express.static setHeaders AND SPA fallback route.

---

## The UI: Two-Tier Pillar Cards

The frontend uses a two-tier display per pillar:

1. **RECENT** (<48 hours) — always visible, full opacity
2. **ALL ARTICLES** — expandable, 30-day window, with age-based opacity fade

Articles older than 48 hours render at reduced opacity. Articles older than 7 days are near-transparent. This gives a visual signal of freshness without hiding older content entirely.

Each pillar card shows:
- Top signal headline (from LLM analysis)
- Five-bullet summary
- Action items
- Full article list with source, date, and matched keywords

The PRIORITY section (tier 1 keyword matches) is collapsible by default — it opens on click. This keeps the default view focused on the LLM summary while letting you drill into the raw articles when needed.

---

## What I'd Do Differently

**Start with source diversity.** The biggest gap wasn't in the filter logic — it was in the source list. Three of my original five RSS sources were from the same niche (crypto news). Adding nomad, security, and AI-specific sources had more impact than any amount of keyword tuning.

**Test RSS URLs before coding around them.** Two of my first four new sources were broken (Cloudflare 403 and HTML-instead-of-XML). A 10-second `curl -sI` check would have caught both.

**Bucket before you score.** If I'd started with the bucketed approach instead of weighted scoring, I would have saved the 2 iterations it took to realize that freshness and relevance are separate axes.

---

## Numbers

| Metric | Value |
|--------|-------|
| RSS sources | 17 (was 10) |
| Articles per day | ~250-400 |
| Articles surfaced | 45 max (15 per pillar) |
| Filter coverage | ~8% of raw articles |
| LLM analysis time | 5-8 seconds (3 parallel calls) |
| Pillar coverage | 3/3 active (was 2/3) |
| Keyword model version | v28+ |
| Priority terms | 38 across 3 pillars |
| Standard keywords | 116 across 3 pillars |

---

## What's Next

The pipeline works. But it's still reactive — I see what happened yesterday, not what's happening now. Next steps:

1. **Breaking news alerts**: Use the webhook endpoint (`POST /api/defi/alert`) to push high-priority articles to the OBOS notification system in real-time
2. **Cross-pillar correlation**: When an article matches multiple pillars (e.g., "AI-powered tax optimization for digital nomads"), surface it in both with a correlation badge
3. **Source quality scoring**: Track which sources produce the most articles that survive the keyword filter. Demote low-signal sources automatically

The goal isn't to read more news. It's to read less, better.

---

*Built with Claude Code agents on the OBOS dashboard. Source code, Living Rules, and session data are all public.*
