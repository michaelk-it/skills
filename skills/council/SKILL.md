---
name: council
description: Trigger this if the user says 'council'.
---

<!-- Adapted from t3dotgg/tstack (MIT) for GitHub Copilot agent mode in VS Code.
Original: https://github.com/t3dotgg/tstack/blob/main/skills/council/SKILL.md
Copilot has no subagents and no delegate_task, so the three judges are three
fresh Copilot chats, one per model. Everything else — three independent
reviews, scores out of 10, rounds, no invented scores — is kept as written. -->

# Council

Use when the user says "council", or when a change (especially a UI page)
needs a second, third, and fourth opinion before it ships.

Before an audit, address outstanding PR comments, fix known issues, and run
relevant checks. Give council the updated implementation; do not defer known
fixes until reviews return. If the user wants to discard the implementation
and explore new ideas, skip fixes to code being replaced.

## The three judges — in Copilot

Copilot cannot spin up subagents, so do the council by hand:

1. Open **three fresh Copilot chats**, one per judge. Do not reuse the
   building chat, and do not let a judge see another judge's score.
2. Switch the model picker in each chat to a different provider:
   - Chat 1: Claude Sonnet 5.5 (or Claude Opus 5.5 for a final polish round)
   - Chat 2: GPT-6.1 Sol
   - Chat 3: Gemini (3.8 Flash or 4 Argon — whichever is available)
3. Paste the **same** request into each chat: the goal, the relevant
   design documents or sketch, the diff or the before/after screenshots,
   and the rubric below. Keep each judge's work read-only — they review,
   they do not change code.
4. If a provider or model is unavailable, report it instead of
   substituting another model in its place.

Do **not** tell the judges the target score. A judge that knows the bar
inflates its score.

## What to ask each judge

For a code review, ask each judge for a score out of 10, plus its top 5
concrete fixes, ranked. For a UI review, score these and give an overall
score out of 10:

- **Layout match:** does it match the sketch / design documents, including
  where each component sits?
- **Clarity:** can a person tell what to do on the page in 5 seconds?
- **Hierarchy:** is the most important thing the most obvious?
- **Consistency:** does it follow the shared library (`epc.common.ui`)
  and the design documents, not a one-off style?
- **Polish:** spacing, sizing, alignment, contrast — clean, not messy?

Use this scale, and put it in the judge prompt so all three use the same bar:

- **10** — layout matches the sketch/design documents exactly, the user
  can tell what to do in 5 seconds, spacing and contrast are clean.
- **6** — the layout is roughly there but messy or unclear.

## Collecting the verdict

Collect the three responses and summarise the useful findings, the
disagreements, and the recommended next steps. Take the **median** of the
three scores as the round score — one outlier should not decide it.

Count each full review pass as one round. A fresh review after fixes
starts a new round.

If the work ends in a pull request, put the results in the PR description,
before the model and harness line at the end:

```markdown
## Council review

Rounds: 2

| Delegate | Round 1 | Round 2 |
| --- | --- | --- |
| Claude Sonnet 5.5 | 7/10 | 9/10 |
| GPT-6.1 Sol | 8/10 | 10/10 |
| Gemini | 7/10 | 9/10 |
```

If a judge gives no score, write `no score`. Do not invent one.
If the PR is open, edit the description. Do not post a comment for this.
If there is no PR, put the same results in the final reply only.

If you agree with the recommendations, make the changes and request a
fresh review after. Continue until all judges are satisfied with the
outcome — or until the scores stop rising. For UI work, stop at 3 rounds
maximum; keep a round only if the score rises.

Do not let repeated reviews result in scope creep. It is your job to
fight to keep scope limited. A judge's 9 still fails if a real person
cannot complete the page's job — the user is the final check.
