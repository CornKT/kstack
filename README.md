# kstack

`kstack` is a collection of agent skills and shared instructions for Claude
Code and Codex. It focuses on controlled implementation, review, documentation,
and pull request workflows.

## Default and explicit modes

Without an orchestration command, the main model handles the request on its
own. It does not run formal task sizing, delegate child work, or start a
multi-model review.

Use an explicit command when you want orchestration:

| Command | Purpose |
| --- | --- |
| `/council` | Run a council review with the eligible configured reviewers. |
| `/sol-loop` | Run a bounded review and fix loop. |
| `/pstack-interrogate` | Run an adversarial multi-model review without auto-applying fixes. |
| `/pstack-blast-radius` | Check what a change could break outside its direct callers. |
| `/full-send` | Implement, prove, open, monitor, and merge a PR. |
| `/full-stop` | Implement, prove, open, and monitor a PR without merging it. |

T3 Code also accepts the skill form, such as `$council` and `$full-stop`. The
command selects the workflow. For `full-send` and `full-stop`, task size sets
the review depth. For review commands, the command fixes the review mode and
task size sets its scope and budget. A command does not silently switch to
another workflow.

Proof requirements, branch protection, and the rule against staging unrelated
changes apply in both modes. In default mode, the main model also runs the
relevant checks and reports risky changes. It suggests an explicit review
command without starting one.

## Skills

### Pull request workflows

| Skill | Purpose |
| --- | --- |
| [`file-pr`](skills/file-pr) | Open a concise pull request with a checked diff and a clear description. |
| [`babysit-pr`](skills/babysit-pr) | Monitor checks, reviews, conflicts, and follow-up changes for one PR. |
| [`full-send`](skills/full-send) | Take an explicit request through implementation, proof, review, monitoring, and merge. |
| [`full-stop`](skills/full-stop) | Take an explicit request to a merge-ready PR without merging it. |
| [`modernize`](skills/modernize) | Reassess and update an aging PR against its current base branch. |
| [`takeover`](skills/takeover) | Continue existing work while preserving contributor credit and intent. |
| [`leaving-pr-comments`](skills/leaving-pr-comments) | Format and verify agent-authored GitHub PR comments. |

Invoke these workflows when you want their side effects. They run only after
you invoke them. A watcher may remain active after `babysit-pr` or `full-send`
starts.

### Review and auditing

| Skill | Purpose |
| --- | --- |
| [`council`](skills/council) | Ask the eligible configured reviewers to compare findings. A council can run with one member when the other wrote the affected code or is unavailable. |
| [`sol-loop`](skills/sol-loop) | Run one configured frontier reviewer through a bounded fix loop. |
| [`pstack-interrogate`](skills/pstack-interrogate) | Run an explicit adversarial multi-model review without auto-applying fixes. |
| [`pstack-blast-radius`](skills/pstack-blast-radius) | Prove what a change could break outside its direct callers. |

Use `/council`, `/sol-loop`, `/pstack-interrogate`, or
`/pstack-blast-radius` when you want these review workflows. A normal request
to fix code stays in the main thread.

### Documentation and writing

| Skill | Purpose |
| --- | --- |
| [`pstack-technical-writing`](skills/pstack-technical-writing) | Write or review README files, RFCs, runbooks, and other technical docs. |
| [`pstack-unslop`](skills/pstack-unslop) | Remove filler, AI wording, and unclear prose before publishing. |

Technical writing applies `pstack-unslop` to every document it edits.

### Effect v4

| Skill | Purpose |
| --- | --- |
| [`effect`](skills/effect) | Guide Effect v4 services, layers, schemas, configuration, schedules, caches, streams, HTTP clients, and tests. |

Effect is selected for Effect v4 work. It does not start a child model by
itself.

## Install the skills

Run this fish script from the repository root. It links every skill into the
Claude Code and Codex skill directories and can be run again safely.

```fish
function __kstack_install
    set -l repo (pwd)
    if not test -d $repo/skills
        echo "Run this script from the kstack repository root." >&2
        return 1
    end
    mkdir -p ~/.claude/skills ~/.agents/skills

    for skill in $repo/skills/*
        test -d $skill; or continue
        set -l name (basename $skill)
        for target in ~/.claude/skills ~/.agents/skills
            set -l link $target/$name
            if test -e $link; and not test -L $link
                echo "Refusing to replace existing directory or file: $link" >&2
                return 1
            end
            ln -sfn $skill $link
        end
    end
end
__kstack_install
functions -e __kstack_install
```

The shared instructions live in [`AGENTS.md`](AGENTS.md). Codex reads it from
`$CODEX_HOME/AGENTS.md`. If `CODEX_HOME` is unset, use `~/.codex/AGENTS.md`.
Claude Code should include the same file from its `CLAUDE.md` configuration:

```text
@/path/to/kstack/AGENTS.md
```

## Repository checks

This repository does not yet ship a validation script or CI workflow. Before
committing documentation or workflow changes, run `git diff --check`, validate
the changed Markdown links, and parse the Python watcher without writing
bytecode:

```fish
git diff --check
python3 -c 'import ast, sys; ast.parse(open(sys.argv[1]).read(), sys.argv[1])' skills/babysit-pr/scripts/pr-wait
```
