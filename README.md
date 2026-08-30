# Skills

[![Validate skills](https://github.com/paulbalaji/skills/actions/workflows/validate-skills.yml/badge.svg)](https://github.com/paulbalaji/skills/actions/workflows/validate-skills.yml)

Portable [Agent Skills](https://agentskills.io/) for reliable software work.

## Available skills

### `deslop-repository`

An evidence-first, deletion-led workflow for simplifying mature repositories
without weakening behavior, security boundaries, public contracts, or
production proof.

Use it for repository de-slopping programs, dead-path removal, duplicate
contract consolidation, architectural simplification, or durable cleanup
guardrails. Its detailed operating guide also includes reusable issue and pull
request checklists plus a copy-paste prompt for agents that do not support the
Agent Skills format directly.

## Install

Clone this repository, then copy or symlink the desired skill directory into
the skill directory recognized by your agent. Skill locations and installation
commands vary by client; the portable unit is the folder containing `SKILL.md`.

For a project-local installation using the standard layout:

```text
.agents/skills/deslop-repository/
├── SKILL.md
└── references/
    └── operating-guide.md
```

See the [Agent Skills specification](https://agentskills.io/specification) for
the format and your agent's documentation for its discovery path.

Every pull request and push to `main` discovers all `SKILL.md` files and
validates each skill directory with the pinned official `skills-ref` reference
library. The workflow fails when no skills are present or any current or future
skill violates the specification.

## License

MIT
