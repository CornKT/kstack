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
- default child target: omit `target`, so the child runs on the parent thread's own provider and model; choose another only when the user names one or a skill says to
- never pick a Gemini model, which rules out every `antigravity` model; run Claude children on T3's `claudeAgent` provider
- use the harness's native subagent tool only when T3 is unavailable; state the reason when you fall back
- never request `max` or `ultra` effort, and never enable fast mode or the `priority` service tier, unless the user instructs it; `xhigh` is the ceiling, and pass `serviceTier=default` explicitly because some models default to `priority` and others default to `low` effort
- check every target above against `orchestrator_capabilities` at call time; if one is missing, report it instead of guessing another

When using kstack skills (`council`, `sol-loop`, `full-send`, and the rest):

- the council, and the `pstack-interrogate` reviewers, are the latest Claude Opus and the latest GPT Sol, each at `xhigh`; resolve the newest version from `orchestrator_capabilities` at call time and never pin one; only frontier-class models sit on it, so never seat GPT Luna or another budget model
- never add a Gemini model to the council, not even as a jester
- `sol-loop` reviews with the latest GPT Sol at `xhigh`, and `full-send` calls it as written; if `orchestrator_capabilities` lacks Sol, `sol-loop` reports the blocker, and in `full-send`, run `council` where it says `sol-loop`

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
