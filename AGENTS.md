# Global agent instructions

Shared by every harness: Claude Code imports this file from `~/.claude/CLAUDE.md`,
and Codex reads it as `$CODEX_HOME/AGENTS.md`. Shared skills live in `~/.agents/skills`.

About the user (Kayti):

- reply in Vietnamese; write code, code comments, commit messages, pull requests, and docs meant for the repository in English unless the repository already uses another language
- the machine runs Linux on WSL2; the login shell is fish, so when you hand the user a command to paste, write it in fish syntax
- `pnpm` is the JavaScript package manager; use the project's lockfile manager when a project already uses another one

When working in typescript:

- when adding a package to a project add it with an install command, instead of manually editing the package json
- run check/format/lint commands when your done making a change. if they don't exist, suggest making them for the project you're in
- avoid explicit return types unless absolutely needed
- `as any` should be an absolute last resort. always use real type safety. lean on type inference instead of manually writing new types over and over again

When showing visuals:

- use Markdown for simple tables, lists, and code; use T3's `html_render` for diagrams, charts, and UI mockups inside a T3 thread

When delegating to subagents:

- prefer the T3 Code orchestrator: call `orchestrator_capabilities` first and delegate with `delegate_task`
- targets below read `<providerInstanceId>/<model> <option>=<value>`; split at the first `/`, and pass the options as `target.options`
- default child target: `codex/gpt-6.1-terra reasoningEffort=xhigh serviceTier=default`; pick other targets from "Model routing" below
- never pick a Gemini model, which rules out every `antigravity` model; run Claude children on T3's `claudeAgent` provider
- use the harness's native subagent tool only when T3 is unavailable; state the reason when you fall back
- never request `max` or `ultra` effort, and never enable fast mode or the `priority` service tier, unless the user instructs it or "Model routing" names it; `xhigh` is the ceiling for every other model, and pass `serviceTier=default` explicitly because some models default to `priority` and others default to `low` effort
- check every target above against `orchestrator_capabilities` at call time; if one is missing, report it instead of guessing another

Model routing:

Default mode uses the main model only. Do not run formal task sizing, delegate
child work, multi-model review, or automatic escalation unless the user invokes
an orchestration command or explicitly asks for delegation. An explicit request
for delegation also runs the size table before choosing a child.

Orchestration commands include `/council`, `/sol-loop`, `/pstack-interrogate`,
`/pstack-blast-radius`, `/full-send`, and `/full-stop`. When one is explicit,
run the size table below to choose its budget, children, proof, and review
depth. For `full-send` and `full-stop`, size selects the review depth. For the
review commands, the command selects the review mode and size only tunes its
scope and budget.

| Size | Typical work | Plan | Implement | Proof | Review |
| :--- | :--- | :--- | :--- | :--- | :--- |
| XS | a typo, copy text, one color or spacing value, a config value, a version bump, a rename the tools can check | none | main thread | the one check that shows it: render it, run it, or grep it | none |
| S | a CSS or layout fix, hover and focus states, a one-function bug with a repro, a new test, docs | none | main thread | reproduce, fix, and watch it pass; for UI, look at it in the browser; run the repo's lint and typecheck | none when the proof covers the whole diff, otherwise a light round |
| M | a fix or feature with real logic across a few files, a new component or endpoint, a refactor inside one module | a plan child only when the request leaves the approach open | main thread under about 150 changed lines, otherwise the implement job | the repo's CI checks plus one focused test for the new behavior | a full round |
| L | a feature across modules, a public API change, a data model change | the plan job | the implement job; hard parts on the hard-part job | CI, tests, and a run of the app | `sol-loop` |

A change is risky when it touches auth or permissions, payments, secrets, data
migration or deletion, concurrency, deploy or infra config, or a hot performance
path. In default mode, the main model still runs the relevant proof and checks,
states the risk, and suggests an explicit review command without starting one.
When `full-send` or `full-stop` is active, risk overrides the default review:
XS and S risky changes use a full round. M risky changes use `sol-loop`. L
risky changes use `council` instead of `sol-loop`. Review commands keep the
requested review mode and do not switch commands because of size. Use the
hard-part job only when the plan marks an algorithm, concurrency, migration, or
security part as hard. Do not run both `sol-loop` and `council` for the same
diff unless the user asks.

Review levels:

- none: the proof shows every changed line working, the diff has no branching logic, or it is a revert, generated output, or formatting only
- light round: one review at `reasoningEffort=high` and no second round; fix any blocker yourself and prove the fix without another review
- full round: one review at `reasoningEffort=xhigh`; continue as `sol-loop` only when it finds a blocker
- `sol-loop`: the skill's own budget, 3 rounds, or 5 for the risky categories defined above
- `council`: the skill's budget, one full round plus one targeted recheck, and a third round for risky work with an open blocker
- the reviewer is never the model that wrote the code; when the main thread runs Sol, review on `claudeAgent/claude-opus-5-5` at the same effort

Keep the overhead smaller than the task:

- a child takes minutes to brief and start, so delegate a job only when it saves more than that: explore only when you would read more than about ten files, and the fix loop only when failures are many or slow
- for XS, S, and M, the main thread writes the PR description itself; for XS and S it does all the other jobs too, and its own model is enough, so a Terra thread suits this work
- time budgets: XS 5 minutes, S 15, M 40, L none; time spent waiting on CI, a human, or a secret does not count; at twice the budget, size the task again, then move it up with a reason or report what blocks it
- when the work turns out bigger or riskier than its size, move it up and say so; never move it down to save time

Pick each child by the job it does. Every row has a primary and a fallback on another provider, so one provider's outage or quota never stops the job. Codex targets all take `serviceTier=default`.

| Job | Primary | Fallback |
| :--- | :--- | :--- |
| Plan: read the request, choose the approach, split the work, mark hard parts | `claudeAgent/claude-opus-5-5 effort=xhigh` | `codex/gpt-6.1-sol reasoningEffort=xhigh` |
| Explore: find and read the relevant code, summarize logs or diffs | `codex/gpt-6-luna reasoningEffort=max` | `claudeAgent/claude-sonnet-5-5 effort=medium` |
| Implement a planned change | `codex/gpt-6.1-terra reasoningEffort=xhigh` | `claudeAgent/claude-sonnet-5-5 effort=high` |
| Implement a part the plan marks hard (algorithms, concurrency, migrations, security) | `codex/gpt-6.1-sol reasoningEffort=high` | `claudeAgent/claude-opus-5-5 effort=high` |
| Fix loop: run tests, lint, and typecheck, and fix what fails | `codex/gpt-6-luna reasoningEffort=max` | `claudeAgent/claude-sonnet-5-5 effort=medium` |
| Review | `codex/gpt-6.1-sol reasoningEffort=xhigh` | `claudeAgent/claude-opus-5-5 effort=xhigh` |
| Write the PR description and docs | `claudeAgent/claude-sonnet-5-5 effort=medium` | `codex/gpt-6.1-terra reasoningEffort=xhigh` |

- switch to the fallback only on an infrastructure failure: the model is missing from `orchestrator_capabilities`, quota or rate limit, or the task crashes; switch once per job, and when the fallback fails too, stop and report both errors
- a weak result is not a reason to switch; tests and review catch it
- the reviewer must never be the model that wrote the code; Sol reviewing Terra or Luna is fine because it is the stronger model, but when Sol or Opus wrote a hard part, review that part on the other one
- name every fallback in the report, with the job and the error, e.g. "implement: Terra rate-limited, ran on Sonnet"

When using kstack skills (`council`, `sol-loop`, `full-send`, and the rest):

- the review roster for `council` and `pstack-interrogate` is `claudeAgent/claude-opus-5-5 effort=xhigh` and `codex/gpt-6.1-sol reasoningEffort=xhigh serviceTier=default`; only frontier-class models sit on it, so never seat GPT Luna or another budget model. Exclude each model from the files or parts it wrote.
- never add a Gemini model to the council, not even as a jester
- `sol-loop` runs on `codex/gpt-6.1-sol reasoningEffort=xhigh serviceTier=default`, or the configured non-writer frontier reviewer when Sol wrote the scoped part; the size table decides whether a task needs it at all

When running shell commands:

- prefer `rg` over `grep` and `fd` over `find`; where a newer replacement for a classic tool is installed (check with `command -v`), use it, and fall back to the classic tool when it is not
- keep scripts committed to a repository on tools every machine has

In general:

- when asking questions, ask them one at a time

When a task needs root:

- `sudo` asks for a password on this machine; tell the user the exact command and what it is for, and let them run it
- never feed a password to `sudo` or `run0`, including one from a secret store, and never bypass the prompt

When you need temporary files:

- keep them in your harness's scratch space, such as Claude Code's session scratchpad; never loose in `/tmp`
- delete what you created before handing back, unless the user needs it; then say where it is
- files there untouched for 3 days are deleted automatically, so never keep anything there that must last

When committing:

- run `git status --short` first; never stage or commit unrelated changes
- in a repository with a GitHub remote, work on a branch and land it through a pull request with a squash merge
- write every commit message and pull request title as a Conventional Commit (`type(scope): summary`, e.g. `feat(services): …`, `fix(ci): …`, `docs: …`); the squash merge uses the PR title as the commit on the default branch
- check the current branch before committing, and never commit or push to the default branch
