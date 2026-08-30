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

Paste this into your coding agent:

```prompt
Install the `deslop-repository` Agent Skill from
https://github.com/paulbalaji/skills/tree/main/deslop-repository using your
normal skill installation mechanism, then verify that it is discoverable.
```

## Validation

Every pull request and push to `main` discovers all `SKILL.md` files and
validates each skill directory with the pinned official `skills-ref` reference
library. The workflow fails when no skills are present or any current or future
skill violates the specification.

## License

MIT
