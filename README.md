# agent-skills

Reusable **skills for AI coding agents** (SKILL.md format, compatible
with OMP managed-skills and Claude-style skill loaders). Each directory is a
self-contained skill: a `SKILL.md` with YAML frontmatter (`name`, `description`) plus any
supporting scripts.

These are distilled from daily, production agent use — every rule in them exists because
a real failure happened first.

## Skills

| Skill | What it does |
|---|---|
| [`typst-it`](typst-it/SKILL.md) | Generate polished, print-quality [Typst](https://typst.app) technical reports — investigation writeups, RCAs, architecture docs — with fletcher diagrams, semantic callouts, ink-light styling, and a strict two-pass (layout **and** content) verification loop. |
| [`blog-it`](blog-it/SKILL.md) | Write and publish a Jekyll + GitHub Pages post safely: frontmatter conventions, link verification before commit, post-deploy liveness checks, and the future-date/timezone trap that silently drops posts. |
| [`ffs`](ffs/SKILL.md) | **Friction → Fix → Safeguard.** Mine recurring user corrections from agent session transcripts, classify whether each is mechanically detectable, and convert the ones that are into tested rules — with positive/negative test cases and a one-week recurrence check. |

## Using a skill

Point your agent's skill loader at the directory, or copy the skill folder into your
skills path (e.g. `~/.omp/agent/managed-skills/<name>/` for OMP). The `description`
frontmatter is what the agent matches against, so keep it when adapting.

## License

[MIT](LICENSE)
