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
