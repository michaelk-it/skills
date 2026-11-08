---
name: full-stop
description: Trigger this if the user says 'full stop', even as a closing "Full stop." that reads like emphasis.
---

<!-- Adapted from t3dotgg/tstack (MIT) for GitHub Copilot agent mode in VS Code.
Originals: https://github.com/t3dotgg/tstack/blob/main/skills/full-stop/SKILL.md
and https://github.com/t3dotgg/tstack/blob/main/skills/full-send/SKILL.md
Full-stop in tstack is a thin wrapper over full-send, so this version is
self-contained: the workflow, proof rules, and merge gate from full-send
are inlined below. It does NOT merge — that is the whole point.
Copilot adaptations: no watch_pull_request / delegate_task — check the PR
with `gh`, and ask the user in plain chat instead of a harness question
tool. The testing permission below is for this task only and does not
override any Copilot approval prompt that appears. -->

# Full Stop

Do the same work as a full end-to-end run, but stop with the PR ready for
the user to merge. **Never merge it.**

An explicit "full stop" request authorizes that work — including the test
methods Proof needs, even when a project file says to ask first, for
example before running the app, a dev server, screenshots, or visual
checks. Use those methods directly. Its merge permission does not apply.

## Steps

1. **Finish the change.** Complete the goal from the user's request with the
   smallest change that works. Fix the case that happened, not every case
   that could. If you are on the default branch, create a branch first.
2. **Prove it.** Get proof for every change you made. See "Proof" below.
3. **Get feedback.** Use the `sol-loop` skill in this repo to verify the
   changes (a fresh Copilot chat switched to GPT-6.1 Sol).
4. **File the PR** with the `file-pr` skill in this repo. Rebase onto the
   latest default branch first. Add one short line to the description that
   says how you proved the change.
5. **Check the PR.** Run `gh pr checks` and `gh pr view`. Fix real findings
   and CI failures, and get proof for each fix before you push it. Verify
   every review-bot finding against the source before changing code; reply
   with a written reason when dismissing a false positive. Do not wait
   indefinitely in one chat for checks to finish — if they are still
   pending, hand off with the PR link and say what is still pending.
6. **Stop at the gate.** Do not merge. Hand off instead (see below).

## Proof

Proof is evidence you observed yourself in this session: command output, a
reproduction, a screenshot. Use the most direct proof the change allows:

- Reproduce the original failure, apply the fix, then watch the same
  reproduction pass.
- Run the checks that the repo's CI runs (typecheck, lint, tests —
  for a .NET backend, `dotnet build` and `dotnet test` must still pass).
- Add one focused test that fails without the fix, when the behavior has
  none and the repo has a test setup.
- For UI and runtime behavior, run the app and look at the change. Green
  mocked tests do not prove that the user experience works. Attach
  screenshots to the PR when they help and you have a way to upload them.

When you cannot find a way to prove a change, you do not have full
confidence. Ask the user.

## The readiness gate

The PR is ready only when all of these are true on the latest commit:

- Your proof covers every change in the diff.
- Required checks are green.
- Review bots have finished, and each finding is fixed or dismissed with a
  written reason.
- No human review still requests changes.
- The branch is up to date with the default branch, and GitHub reports it
  as mergeable.

## No merging

You may commit, push, rebase, fix conflicts, and use `--force-with-lease` on
this task's branch. Never merge, enable auto-merge, enter a merge queue,
push to the default branch, bypass branch protection, or delegate those
actions.

Disable existing auto-merge and remove the PR from any merge queue before
work begins; check again before each push. Report these changes. If you
cannot prevent automatic merging, report the blocker before proceeding.

## Handoff — instead of merging

Fetch the target branch and check the live PR's head, base, checks,
reviews, and mergeability. Confirm it is open, not a draft, and not set to
merge without the user. If someone else merged it, report that instead.

Give one verdict:

- **Ready to merge:** the gate passes. Leave the PR open and unmerged.
- **Waiting for required approval:** only human approval remains. Confirm
  that is the actual blocker.
- **Blocked:** explain what prevents readiness, what you tried, and what
  is needed.

Report the PR link, final head, verdict, proof, reviewer outcome, and
remaining risks. State whether the PR is still open and unmerged.

Ask the user only when you cannot continue alone: you cannot prove the
change, the gate cannot pass and you cannot fix that yourself, the next
step is outside the grant above (deleting data, touching production,
changing things outside the repo, closing a PR), or the goal is ambiguous
in a way that changes the result. Ask in plain chat, with the PR link,
what blocks you, and the options with your recommendation first. Decide
everything else yourself with sensible defaults, and list those decisions
in the handoff.
