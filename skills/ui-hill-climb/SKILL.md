---
name: ui-hill-climb
description: Trigger this if the user says 'hill climb', 'ui hill climb', or asks to improve a page or backend area by scoring it and iterating in rounds.
---

<!-- The user's own hill-climb process, built from Theo Browne's method
(T3 Code) and refined on his React + .NET internal/external dashboards
with Copilot agent mode in VS Code. Copilot adaptations: Claude Sonnet
5.5 is required because it is a Vision model — do not use a model
without Vision. Copilot has no subagents, so judges are fresh Copilot
chats (see council in this repo). The builder chat scores and fixes;
judges review only. -->

# Hill Climb

Improve one page (or one backend area) by scoring it, fixing the worst
problems, and re-scoring — keep a round only if the score goes up, stop
at 9 or above, max 2 rounds per page (3 for a first build). The user is
the final check, not the score.

## Before starting

- **One page at a time.** Finish one page completely before the next.
  If several pages are in scope, first list every page in both
  dashboards, with its route. That list is the work list.
- **Score every page fresh.** A score from one page never carries to
  another. Do not assume any page is already at 8 or 9.
- **The current implementation is the source of truth for layout.**
  Do not revert anything back to the initial sketch or design doc —
  the sketch is only a rough reference now. Score the page against
  how it looks and works today, and only fix what is actually broken.
- **Read the shared library first** (`epc.common.ui` or the project's
  equivalent): follow its structure, patterns and tokens, reuse its
  components first, do not build duplicate one-off versions. If a new
  component will be reused, add it to the shared library and export
  it. If you change a shared component, re-check every page that uses
  it before moving on, and do not break the other app.
- **Do not change the API, routes, or backend.** `dotnet build` and
  `dotnet test` must still pass for a round to be kept.

## UI: design first, then responsive

Do design first — design changes touch every size, so responsive work
done first has to be redone.

1. **Design.** Improve where it clearly makes sense: hierarchy,
   spacing, sizing, alignment, typography, colour and contrast,
   readability, how clear the main action is. Do not add features.
   Do not move components unless moving one clearly makes the page
   easier to understand, and say why if you do. Screenshot at 1440px
   and 390px with real backend data. Score each 1-10 (rubric below).
   Fix the top 5 issues. Max 2 rounds.
2. **Responsive.** Check the same page at 1920, 1440, 1280, 1024,
   768, 390, and 360px with Playwright screenshots (full page and
   above the fold), real backend data. At each size check: no
   horizontal scroll or overflow, no clipped or overlapping text or
   buttons, tables and cards adapt instead of breaking, navigation
   still works, touch targets big enough on mobile, text readable,
   spacing and contrast clean. On desktop keep components in their
   original locations; stacking and wrapping on tablet/mobile only
   where the design documents or the shared library's breakpoints say
   so. Fix the top 5 issues across all sizes, worst size first.
   Max 2 rounds. Keep a round only if the **lowest** score goes up.
   Stop at 9 or above on every size, or keep the best version.

## UI rubric (same bar every round)

- **10** — I can tell what to do in 5 seconds, hierarchy and spacing
  are clean, contrast is good, nothing overflows.
- **6** — the layout is roughly there but messy or unclear.

For a sharper judge pass, number the top 5 problems directly on the
screenshot and list the fixes by the same number, with one overall
1-10 score. Do not tell a judge the target score — a judge that knows
the bar inflates it.

## Backend: same loop, numeric score

For backend the judge is numbers, not vibes — do not let an AI judge
score code quality 1-10.

1. **Define the score first and run a baseline:** how many
   `dotnet test` tests pass, and how many of these API cases pass —
   valid request, invalid input, missing record, auth failure, slow
   query (add cases that matter for this endpoint). Note the baseline
   numbers.
2. Fix the top 5 failures or weak spots. Max 3 rounds. After each
   round re-run the tests and the API cases, and keep the round only
   if the total score goes up. Stop when all tests and cases pass, or
   when a round adds nothing.
3. Use a fresh-chat reviewer only for readability and design, never
   as the score.

## Report

After each page, a short summary for that page only, then move to the
next. At the end: a table with every page, its before/after scores at
1440px and 390px (and every size for responsive), and what changed on
it. For backend: before/after test count, before/after case count, and
what changed and why.
