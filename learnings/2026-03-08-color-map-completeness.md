---
date: 2026-03-08
source: reflect
issue: "#335"
tags: [debugging, patterns]
blog_worthy: false
---

# Color maps need corpus verification, not just parser audits

## What Happened
A brief rendering feature was shipped with a parser fix but was reported as "not working." The parser was correct — the bug was a missing key in a 21-entry color lookup map. TEACH (used in one old-format brief) fell back to gray, making the entire feature look broken.

## The Insight
When a rendering pipeline has a data-driven lookup (color maps, label maps, icon maps), the completeness of the map matters more than the correctness of the parser. Always cross-reference map keys against the full corpus of real data.

## By The Numbers
- 21 existing SECTION_COLORS entries, 1 missing (TEACH)
- 18 briefs in corpus, 3 distinct formats requiring different parser paths
- 1-line fix after thorough audit

## Pattern
Corpus-driven map validation: enumerate all unique values from real data, assert each has a map entry.
