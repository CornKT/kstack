---
name: pstack-interrogate
description: 'Use when the user explicitly invokes `/pstack-interrogate` or `$pstack-interrogate` for an adversarial multi-model review.'
metadata:
  harness: [claude, codex]
  platform: [linux]
  scope: fleet
  origin: "https://github.com/chhoumann/t3-pstack (238d156), Lauren Tan's pstack, MIT"
  requires: "T3 Code delegation; review roster from the active global instructions"
---

# Interrogate

Spawn one reviewer per configured model to adversarially review code changes. The adversarial signal comes from model diversity, not assigned personas. The deliverable is a synthesized verdict. Do NOT auto-apply changes.

## Step 1, Scope

Identify what to review from context:

- If the user points at specific files or a diff, use that.
- If on a feature branch, run `git diff main...HEAD` (or the appropriate base branch) for the full changeset.
- If the user's message references recent work, gather the relevant files.

Package the diff or file contents with any surrounding context files the reviewers need to understand the code.

## Step 2, Intent

Before spawning reviewers, state the intent in one clear paragraph from the user's message, commit messages, the PR description if one exists, and the code. If you're unsure about the intent, ask the user before proceeding.

## Step 3, Spawn Reviewers

Call `orchestrator_capabilities` first in T3 Code. Launch all reviewers in a
single response with `delegate_task`, one per model in the review roster from
the active global instructions, labeled Reviewer A, B, and so on to match the
roster. Do not assign a model to review the files or parts it wrote. If no
roster or eligible reviewer is available, report that instead of choosing
models.

Each reviewer gets `role: "review"`, a brief that says to inspect only and
change no file, and `target` set from its roster entry.

If a roster entry is unavailable, continue with the remaining eligible members
and name the gap. With none left, stop and report. Do not lower the model tier,
seat a duplicate roster member, or silently change provider.

Read `references/reviewer-prompt.md` and fill in the same template for every reviewer with the stated intent, the diff or file contents, the rubric from `references/rubric.md`, and the code-quality lens from `references/code-quality-review.md`. Use each roster entry's reasoning level as written; never raise it to `max` or enable fast mode.

## Step 4, Synthesize

As results come back, build a unified picture. Parse every reviewer's findings. Merge findings that describe the same issue differently, and note which models raised each one. Findings two or more models raised independently are the highest signal. Read a lone model's finding, but weight it accordingly. Note disagreements. If one model flags something and another explicitly says the opposite, that is context for the verdict.

## Step 5, Lead Judgment

You are the lead reviewer, a pragmatic senior engineer, not a neutral aggregator. Read `references/lead-judgment.md` for the full framework. Put every finding in one of four buckets, with the model(s) that raised it and a one-line rationale.

- **Act on**. Real issues affecting correctness, security, or maintainability given the actual goals. They would block a real PR.
- **Consider**. Legitimate, but you're not sure they outweigh the cost of addressing them now. Worth the user's attention.
- **Noted**. Technically valid but not actionable. Context-dependent, premature optimization, or low-impact at this stage.
- **Dismissed**. Wrong, nitpicky, or missing context, with a brief reason.

## Output Format

### Intent
> [The stated intent paragraph from Step 2]

### Reviewers
- Reviewer [label]: [model name], [N findings] (one bullet per reviewer)

### Act On
[Each: description, which models raised it, why it matters.]

### Consider
[Each: description, which models raised it, the tradeoff.]

### Noted
[Brief list.]

### Dismissed
[Each with a brief rationale.]

### Agreement Map
[Where models agreed and diverged, and what that pattern tells us.]
