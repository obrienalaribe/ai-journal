---
date: 2026-03-08
source: refactor
issue: "#318"
tags: [dead-code, architecture, patterns]
blog_worthy: false
---

# De-exporting internal helpers after route file splits

## What Was Found
After splitting a 3976-line Express route file into 4 modules (helpers + 3 routers), knip flagged 8 exports in the new shared helpers module that were used internally but never imported by any consumer. These accumulated because the extraction script exported everything that was previously module-scoped.

## What Was Removed
- 5 constants de-exported (CONTENT_SOURCES_DIR, SCRIPT_VISIBLE_STATUSES, VALID_SCRIPT_PILLARS, PIPELINE_STATUS_ORDER, VIDEO_ONLY_STATUSES)
- 2 functions de-exported (resolveTranscriptAssetPath, normalizeTranscriptSourceAsScript)
- 1 re-export removed from convenience line (PROJECT_ROOT_DIR)

## By The Numbers
- 8 unnecessary exports → 0
- Build verified clean after each de-export
- Total route module LOC: 3976 → 3550 across 4 files (content-helpers.js 723 + content.js 1099 + photos.js 217 + schedule.js 1511)

## Pattern
After any file split, run knip immediately to catch over-exported internal symbols. The extraction defaults to `export` for safety — post-split audit narrows the public API surface.
