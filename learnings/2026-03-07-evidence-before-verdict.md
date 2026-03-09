---
date: 2026-03-07
source: reflect
issue: "#300"
tags: [ci, testing, patterns]
blog_worthy: false
---

# Evidence capture must run before the judge, not after

## What Happened
Our CI pipeline captured static screenshots after the reviewer had already approved. This meant the reviewer never saw whether features actually rendered data or just showed empty states. We moved evidence capture (screenshots + API interception + interaction clicks) to run before the review step.

## The Insight
Quality gates are only as good as the evidence they see. If your proof-of-work runs after the approval decision, the decision is made blind. Structure your pipeline so evidence is an input to review, not a post-hoc artifact.

## By The Numbers
- Evidence artifacts: 2 types (before/after screenshots) to 5 types (+ interaction screenshot, API call log, QA report)
- Pipeline position: moved from step 6 (post-judge) to step 5 (pre-judge)

## Pattern
Evidence-as-input: collect proof before the review gate, not as a post-ship afterthought.
