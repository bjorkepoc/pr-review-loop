# PR Review Loop

PR Review Loop is a Codex skill for reviewing, securing, and fixing a pull
request repeatedly until the complete PR is clean on one exact commit.

Its short command is `$prrloop`.

It prevents a common false finish: fixing one round of findings, then calling
the PR clean without re-reviewing the changed result. Each round freezes the
remote SHA, validates findings, fixes root causes, runs relevant checks,
publishes the fix, and starts fresh reviews against the new SHA.

## What `CLEAN` means

- A dedicated security review found no reachable P0-P3 defect.
- All other independent review tracks found no P0-P3 defect on the same SHA.
- Confirmed findings have regression proof.
- Relevant local checks and required CI pass.
- The PR head is the reviewed and tested commit.
- No actionable review thread remains unresolved.

Missing evidence is reported as `BLOCKED`, not `CLEAN`.

## Security is mandatory

Every round includes a dedicated security track. It follows changed trust
boundaries from attacker-controlled inputs through authorization and other
controls to sensitive sinks. Coverage includes authentication, tenant
isolation, injection, secrets, privacy, migrations, data integrity,
concurrency, dependencies, and deployment controls where relevant.

The review remains PR-scoped: unchanged code is inspected only when needed to
prove whether a changed path is safe.

## Install

```sh
mkdir -p ~/.codex/skills
git clone https://github.com/bjorkepoc/pr-review-loop.git ~/.codex/skills/prrloop
```

Start a new Codex task after installation so the skill catalog reloads.

## Use

Invoke it directly:

```text
$prrloop review PR #123, fix every confirmed finding, and loop until clean
```

Or ask naturally:

```text
Run a review loop on this PR. Find and fix defects, then rerun independent
reviews until every track is clean on the same final commit.
```

The skill uses the repository's own instructions, checks, and commit
conventions. It can work with any language or test stack. Independent agents
and live GitHub access improve coverage; without them it reports the limitation
instead of overstating the result.

## Safety

PR Review Loop preserves unrelated work and stays on the existing PR branch. It does
not create branches, rewrite history, force-push, merge, resolve threads, or
post comments without explicit authorization.

## License

[MIT](LICENSE)
