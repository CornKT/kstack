---
name: council
description: Use when the user explicitly invokes `/council` or `$council`.
metadata:
  harness: [claude, codex]
  platform: [linux]
  scope: fleet
  origin: "https://github.com/t3dotgg/tstack (1085050), Theo Browne, MIT"
  requires: "T3 Code's orchestrator (delegate_task)"
---

# Council

## Budget

- Run one full round and at most one targeted recheck by default.
- Allow a third round only for the risky categories defined in the global
  instructions when a blocker remains, or when the user asks.
- Later rounds inspect changed code and open findings only.
- The lead reviewer decides when the evidence is sufficient; delegates do not
  need to agree unanimously.

Before the first round, run relevant checks. For apply-and-review, also
address PR comments and fix known issues. For explicit review-only or audit
requests, record them without editing the implementation. If the user wants to
discard the implementation and explore new ideas, skip fixes to code being
replaced.

Treat `full-send`, `full-stop`, and any request to implement or ship as
apply-and-review. Only an explicit review-only or audit request is read-only.

Bring in one delegate for each model in the configured review roster, usually
for review or feedback. The current kstack roster has two frontier models:
Claude Opus and GPT Sol. If the user's instructions name council members, use
those. Otherwise use the configured roster after checking its availability.

In T3 Code, use `orchestrator_capabilities` to resolve the models, then
`delegate_task` to start one delegate per eligible roster entry in parallel.
Elsewhere, use the harness's subagent tools. Continue with eligible members
when another member is unavailable and name the gap. If none remain, stop and
report. Do not seat a fallback that is already in the roster.

Do not assign a model to review the files or parts it wrote. It may review
other parts when the diff separates cleanly. If no eligible roster member
remains, report that the review cannot satisfy the separate-reviewer rule.

Give each delegate the request and relevant context. For review or feedback,
keep their work read-only. Collect their responses and summarize the useful
findings, disagreements, and recommended next steps.

For a code review, ask each delegate for a score out of 10.
Count each full review pass as one round.
A fresh review after fixes starts a new round.

If an apply-and-review workflow ends in a pull request, put the results in the
PR description. Put this section before the model and harness line at the end:

```markdown
## Council review

Rounds: 2

| Delegate | Round 1 | Round 2 |
| --- | --- | --- |
| Claude Opus | 7/10 | 9/10 |
| GPT Sol | 8/10 | 10/10 |
```

If a delegate gives no score, write `no score`. Do not invent one. For a
review-only audit, put the results in the final reply and do not edit the PR.

If the workflow allows edits and you agree with a recommendation, make the
smallest change and request a targeted recheck within the budget. The lead is
the orchestrating thread. It may decline a blocker or should only with written
evidence such as a file:line, test, or command output. Stop when no delegate
blocker or should remains open without an evidenced rebuttal. A disputed item
after the recheck goes to the user. Budget end is not approval. An open
blocker/should, or a fix made after the last allowed round, goes to the user.

Do not let repeated reviews result in scope creep. It is your job to fight to
keep scope limited.
