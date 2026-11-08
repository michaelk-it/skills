# My AI Skills

Custom agent skills, installable the Matt Pocock way:

```bash
npx skills@latest add michaelk-it/skills
# or pick one:
npx skills@latest add michaelk-it/skills --skill local-saas-monorepo-blueprint
# list what's available:
npx skills@latest add michaelk-it/skills --list
```

How it works: the `npx skills` installer downloads this public repo, reads `skills/<slug>/SKILL.md`, and copies the skills you pick into your agent's skills folder on your machine. No registry, no account, no cost. To update, run `npx skills@latest update`.

## Layout

```
skills/<slug>/SKILL.md        # YAML frontmatter: name + description (one line), then the skill
skills/<slug>/references/     # optional supporting docs
```

## Skills

| Skill | What it does |
|---|---|
| `local-saas-monorepo-blueprint` | Scaffold or evolve a TypeScript SaaS monorepo in the RecPlus style — Turborepo + pnpm, Hono API with orval-generated per-client SDKs, Drizzle multi-tenant DB, strict TS conventions, and AI instruction files. |
| `council` | Get three independent reviews — Claude, GPT Sol, and Gemini, in three fresh Copilot chats — with scores out of 10 and a median verdict. The judge loop for UI hill-climbing and code review. |
| `sol-loop` | Have GPT-6.1 Sol review a change in a loop (max 3 rounds) before filing a PR, with a findings ledger and a push-back rule. |
| `full-stop` | Finish a change end-to-end — prove it, review it, file the PR — and stop with the PR ready for you to merge. Never merges. |
| `file-pr` | File a concise PR that leads with the problem, not an inventory of changes. |
| `grill-with-docs` | A relentless interview to sharpen a plan or design before building, which also builds a glossary and ADRs as you go. |
| `diagnosing-bugs` | A discipline for hard bugs: build a tight, red-capable feedback loop first, then reproduce, minimise, hypothesise, and fix. |
| `tdd` | Test-driven development: red-green-refactor, tests at agreed public seams only, with good/bad test examples. |

`council`, `sol-loop`, `full-stop`, and `file-pr` are adapted from Theo Browne's tstack repo (MIT) for GitHub Copilot. `grill-with-docs`, `diagnosing-bugs`, and `tdd` are adapted from Matt Pocock's skills repo (MIT) for GitHub Copilot. Each adapted skill says so at the top of its SKILL.md.
