---
name: grill-with-docs
description: A relentless interview to sharpen a plan or design, which also creates docs (ADR's and glossary) as we go.
---

<!-- Adapted from mattpocock/skills (MIT) for GitHub Copilot agent mode in VS Code.
Originals: https://github.com/mattpocock/skills/blob/main/skills/engineering/grill-with-docs/SKILL.md
(in the source repo this is a thin stub that calls the "grilling" and
"domain-modeling" skills; both are inlined here so this skill is
self-contained in Copilot, which has no Skill tool).
Copilot adaptations: "dispatch a sub-agent" becomes "look it up yourself
in the repo first"; questions are asked in plain chat, in rounds, and
the user answers before the next round. -->

# Grill With Docs

A relentless interview to sharpen a plan or design — and a paper trail
left behind as you go. Use it **before** building anything non-trivial:
a new dashboard page, a change to `epc.common.ui`, a new feature, or any
plan where a vague instruction would make Copilot guess a generic layout.

This is the skill behind the lesson that good descriptive instructions
are crucial. Do not build until the grilling is done and the user has
confirmed a shared understanding.

## Part 1 — Grill (the interview)

Interview the user relentlessly until you reach a shared understanding.
Map the plan as a **design tree**: every decision branches into the
decisions that hang off it.

Work the tree in **rounds**. The **frontier** is every decision whose
prerequisites are already settled: the questions you can ask _now_
without guessing at answers you haven't heard yet. Ask the whole frontier
in one round: number each question and give your recommended answer.
Then wait for the user's answers before the next round.

Format a round like so:

```
❓ **Q1** - **<question title>**: <question body, might be multiple paragraphs, including multiple choices>

➡️ <your recommended answer>

---

❓ **Q2** - **<question title>**: <question body, might be multiple paragraphs, including multiple choices>

➡️ <your recommended answer>
```

Word each question so "yes" accepts your recommended answer.

Each round the user answers reshapes the tree: settled decisions push the
frontier outward and unblock questions that depended on them. Recompute
the frontier and ask the next round. A question whose answer depends on
another question still open in this round belongs to a _later_ round,
not this one.

Finding _facts_ is your job, never the user's. When a frontier question
needs a fact from the environment (the repo, the design documents, a
sketch, the existing code), look it up yourself first; don't ask the user
for anything you could find. Don't block on it: a running lookup is an
unsettled prerequisite, so only the questions downstream of it wait;
ask the rest of the frontier now. The _decisions_ are the user's: put
each to them and wait.

The session is done when the frontier is empty: every branch of the
design tree visited, nothing left silently assumed. Do not act on it
until the user confirms you have reached a shared understanding.

## Part 2 — Domain modeling (the paper trail)

While you grill, actively build and sharpen the project's domain model:
challenge terms, invent edge-case scenarios, and write the glossary and
decisions down the moment they crystallise.

### File structure

Most repos have a single context:

```
/
├── GLOSSARY.md
├── docs/
│   └── adr/
│       ├── 0001-event-sourced-orders.md
│       └── 0002-postgres-for-write-model.md
└── src/
```

If a `GLOSSARY-MAP.md` exists at the root, the repo has multiple
contexts. The map points to where each one lives.

Create files lazily: only when you have something to write. If no
`GLOSSARY.md` exists, create one when the first term is resolved. If no
`docs/adr/` exists, create it when the first ADR is needed. Formats are
in `references/glossary-format.md` and `references/adr-format.md`.

### During the session

- **Challenge against the glossary.** When the user uses a term that
  conflicts with the existing language in `GLOSSARY.md`, call it out
  immediately. "Your glossary defines 'cancellation' as X, but you seem
  to mean Y. Which is it?"
- **Sharpen fuzzy language.** When the user uses vague or overloaded
  terms, propose a precise canonical term. "You're saying 'account': do
  you mean the Customer or the User? Those are different things."
- **Discuss concrete scenarios.** Stress-test relationships with specific
  scenarios. Invent scenarios that probe edge cases and force precision
  about boundaries between concepts.
- **Cross-reference with code.** When the user states how something
  works, check whether the code agrees. If you find a contradiction,
  surface it: "Your code cancels entire Orders, but you just said partial
  cancellation is possible. Which is right?"
- **Update `GLOSSARY.md` inline.** When a term is resolved, update
  `GLOSSARY.md` right there. Don't batch these up. `GLOSSARY.md` should
  be totally devoid of implementation details. Do not treat it as a
  spec, a scratch pad, or a repository for implementation decisions.
  It is a glossary and nothing else.
- **Offer ADRs sparingly.** Only offer to create an ADR when all three
  are true: (1) **hard to reverse** — the cost of changing your mind
  later is meaningful; (2) **surprising without context** — a future
  reader will wonder "why did they do it this way?"; (3) **the result
  of a real trade-off** — there were genuine alternatives and you picked
  one for specific reasons. If any of the three is missing, skip the ADR.
