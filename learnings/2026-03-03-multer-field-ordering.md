---
date: 2026-03-03
source: reflect
issue: "#227"
tags: [architecture, debugging, patterns]
blog_worthy: false
---

# Multer Discards Body Fields During File Upload Callbacks

## What Happened
Implemented a photo upload endpoint using multer's `diskStorage` where the destination directory depended on `req.body.category`. Uploads landed in `uncategorized/` because multer processes file fields before text fields in multipart payloads. Fixed by saving to a staging directory first, then renaming after the full body is parsed.

## The Insight
When using multer with dynamic destination paths, never trust `req.body` in the `destination` callback. Multipart form data is streamed in field order, and file fields often arrive before text fields. Use a staging directory pattern: upload to temp, validate, then move to final location.

## By The Numbers
- 2 iteration cycles saved with staging pattern documented as Living Rule
- 6 photo endpoints + 1 new tab (471 LOC) implemented in one session

## Pattern
Staging-then-rename for dynamic-destination file uploads
