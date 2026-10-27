# prrloop

A skill for coding agents that loops on a GitHub pull request. It reviews the
PR, fixes what the reviews find, pushes, and then reviews the whole PR again.
It keeps going until one commit passes everything, or it stops and tells you
what's in the way.

I made it because agents kept doing one round of fixes and calling the PR done
without looking at the result again. Fixes can break things too, so every push
gets a fresh review of the whole PR.

## What counts as clean

You get `CLEAN` only when all of this holds for the same commit:

- the security review (it runs every round, even on small PRs) and the other
  reviews (correctness, tests and build) have nothing open
- every finding and review comment is fixed, rejected with a reason, or left
  because you said so
- every fix has a test, or a repro that was run before and after the fix
- the relevant local checks pass and required CI is green
- that commit is still the PR head

Findings are graded P0 to P3 (see [SKILL.md](SKILL.md)). The agent checks each
one itself and rejects the ones that don't hold up, with the reason in the
report. Style nitpicks don't count. If something can't be checked, or the loop
stops making progress, you get `BLOCKED` and the reason.

`CLEAN` isn't an approval. A person should still review the PR before it's merged.

## Install

```sh
git clone https://github.com/bjorkepoc/pr-review-loop.git ~/.codex/skills/prrloop
```

If it doesn't show up, start a new session. Update it with
`git -C ~/.codex/skills/prrloop pull`. Other agents that load `SKILL.md`
folders can use it too, just clone it into their skills folder instead
(often `~/.agents/skills`).

You need `gh` logged in, or some other way for the agent to read the PR and its
checks. Without push access to the PR branch it can still make the fixes
locally, but it stops at `BLOCKED`.

## Usage

```text
$prrloop PR #123
```

Or just ask for it:

```text
Loop on this PR. Fix what you find and keep re-reviewing until it's clean.
```

It uses the repo's own instructions, test commands and commit style, so it
works with any language or test setup. If the agent can't start subagents, it
runs the reviews one after another and says so in the report.

## What it won't do unless you ask

It works on the existing PR branch (in a separate worktree if your checkout is
on another branch) and leaves your uncommitted changes alone. It won't create
other branches, rewrite pushed history, force-push, merge, mark the PR ready,
resolve threads, dismiss reviews or post comments.

## License

[MIT](LICENSE)
