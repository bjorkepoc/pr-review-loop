---
name: prrloop
description: Repeatedly review, secure, and fix a pull request until a mandatory security track, independent reviewers, relevant tests, required CI, and live PR state are all clean on the same final commit. Use for review loops, fix-and-rereview requests, or instructions not to stop until no findings remain.
---

# PR Review Loop

Run an evidence-backed review, fix, and verification loop against one pull
request. A clean result belongs to one exact remote commit, not to a branch name
or an earlier round.

## Outcome

Stop only when the dedicated security track and all other required review
tracks independently report `CLEAN` on the same final SHA, the remote PR
contains that SHA, relevant local checks pass, required CI is green, and no
unresolved actionable review thread remains.

If a required check cannot run, an external service is broken, or a product
decision is needed, report `BLOCKED`. Never rename incomplete evidence to
`CLEAN`.

## 1. Resolve And Freeze Scope

1. Confirm the repository, current branch, worktree, remotes, and PR number.
2. Fetch remote refs and read live PR metadata, reviews, threads, and checks.
3. Resolve exact base and head SHAs. Review the PR diff from its merge base;
   do not infer scope from commit titles or review unrelated branch history.
4. List staged, unstaged, and untracked files separately. Preserve unrelated
   user work and keep it out of fixes and commits.
5. Record the candidate SHA and changed paths before starting a round.

Stay on the existing PR branch. Do not create or switch branches, rewrite
history, force-push, or merge the PR unless the user explicitly authorizes that
specific action. Use an isolated worktree only when it is necessary to protect
unrelated local work, and explain why first.

## 2. Choose Independent Review Tracks

Select tracks from the actual diff and repository contracts. Every round must
include a dedicated security review whose depth scales with the reachable risk.
Do not omit it for a small PR.

The security track follows changed inputs and trust boundaries through existing
controls to their sinks. Cover what applies:

- authentication, authorization, tenant isolation, and least privilege
- input validation, injection, unsafe deserialization, and command execution
- secrets, sensitive logging, privacy, retention, and data exposure
- migrations, row-level security, integrity, concurrency, and fail-closed behavior
- dependencies, build and deployment controls, and supply-chain exposure

Keep it PR-scoped: inspect unchanged code only when needed to prove whether a
changed path is safe. Do not silently turn the loop into a whole-repository
audit.

Other tracks normally cover:

- behavior and product correctness
- tests, migrations, compatibility, build, deployment, and CI
- minimality and maintainability

Give each reviewer the same base SHA, head SHA, changed paths, repository
instructions, and user acceptance criteria. Reviewers are read-only: they do
not edit, stage, commit, push, or resolve threads.

Keep the reviews genuinely independent. Do not seed a final reviewer with the
earlier rationale or tell it which result to confirm. When agents are
unavailable, run the tracks sequentially and state that independence was
limited; do not pretend multiple reviewers ran.

## 3. Finding Contract

Accept a finding only when it contains enough evidence to act:

- severity or priority
- exact file and tight line range
- concrete failing behavior or missing invariant
- reachable input, state, or caller and the affected sink or outcome
- why existing controls do not close it
- smallest complete fix boundary
- a focused reproduction or test when feasible

Suggestions, speculative hardening, style preferences, duplicates, and gaps
outside the PR's changed behavior are not defects. Record why they were
suppressed instead of expanding scope silently.

For a security finding, include the attacker-controlled source or required
state, the relevant control, the reachable sink, concrete impact, and why the
control fails. Do not report a theoretical weakness without a reachable path.

`CLEAN` means the reviewer completed its assigned coverage and found no P0-P3
defect. Silence, timeout, partial coverage, or “looks good” is not `CLEAN`.

## 4. Validate Before Editing

The coordinating agent owns validation and all writes.

1. Reproduce or statically trace each candidate through the real boundary.
2. Inspect direct callers, siblings affected by the same changed helper, and
   the closest existing control.
3. Dedupe overlapping reports.
4. Classify each candidate as confirmed, suppressed with evidence, or blocked
   by missing proof.
5. Fix only confirmed findings. Do not patch a nearby weakness when the
   reported path is already safe.

For a failing CI job, read the actual failing step and logs. Treat it as a
finding only after identifying a repository cause. Retry without a code change
only when evidence shows a transient infrastructure failure.

## 5. Fix The Root Cause

For every confirmed finding:

1. Add the smallest regression check that fails on the broken behavior when
   feasible, including a legitimate control case.
2. Fix the shared enforcement boundary or root cause, not only the named
   symptom or caller.
3. Reuse repository-native helpers and patterns. Avoid new abstractions,
   dependencies, and unrelated cleanup.
4. Preserve supported behavior, error semantics, migration compatibility,
   tenant isolation, least privilege, fail-closed behavior, and user data.
5. Run the focused reproducer immediately after the change.

If two findings share one root cause, use one fix and prove both paths. Keep a
short round ledger mapping every confirmed finding to its fix and proof.

## 6. Verify And Publish The Round

Run checks in this order:

1. diff hygiene and syntax or formatting
2. focused regression tests
3. owning package or subsystem tests
4. repository-wide checks proportional to blast radius
5. build, migration, integration, and repository-native security checks
   required by the PR

Inspect the final diff for accidental files. Commit only intentional changes
using the repository's commit convention, push normally to the existing PR
branch, and verify that local HEAD, remote branch HEAD, and PR head SHA match.

Wait for required CI. A local pass is not remote CI proof. Do not merge the PR,
mark it ready, resolve threads, or post comments unless the user requested that
write action.

## 7. Start A Fresh Review Round

After every pushed fix:

1. fetch the updated PR
2. freeze the new remote head SHA
3. rerun every required review track against the complete PR result
4. use fresh reviewers when available
5. recheck live review threads and required CI

Never carry a `CLEAN` verdict from an older SHA forward. Review the complete PR
result, not only the last corrective commit.

If a new finding appears, return to validation and continue the loop. If the
same failure survives two attempted fixes, stop retrying the same approach and
retrace the root cause before editing again.

## Closure Gate

Declare success only when one immutable final SHA satisfies every item:

- the dedicated security track explicitly returned `CLEAN`
- all other planned review tracks explicitly returned `CLEAN`
- every earlier confirmed finding has a fix and regression proof
- no unresolved actionable PR thread or requested change remains
- focused and repository-required local checks pass
- all required GitHub checks pass
- PR head equals the reviewed and tested SHA
- worktree is clean, or unrelated remaining files are named explicitly
- PR is still unmerged unless the user separately requested merge

## Progress And Final Report

Keep round updates compact:

```text
Round N @ <sha>
Findings: <confirmed / suppressed / blocked>
Fixes: <files or none>
Checks: <pass / fail / running>
Next: <specific action>
```

The final response must state the PR, base and final head SHA, number of rounds,
findings fixed, commands and CI results, review-track verdicts, worktree and
remote state, and whether the PR is draft, ready, or merged.
