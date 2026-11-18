# Plugins & Slash Commands

Nexus loads compatible plugins that expose slash commands to the operator.

## Installed Plugins

| Plugin            | Version       | Command            | Purpose                            |
|-------------------|---------------|--------------------|------------------------------------|
| caveman           | v0d95a81d35a9 | `/caveman`         | Activate caveman plugin guidelines |
| ruflo-autopilot   | v0.2.0        | `/ruflo-autopilot` | Activate autopilot guidelines      |
| ruflo-core        | v0.2.0        | `/ruflo-core`      | Activate core guidelines           |
| ruflo-swarm       | v0.1.0        | `/ruflo-swarm`     | Activate swarm guidelines          |

## Skill Staging

New specialized skills are synthesized into a **quarantined staging folder**:

```
.nexus/staging/skills/<kebab-case-name>/SKILL.md
```

A skill consists of:
- `name` — kebab-case identifier (e.g. `benchmark-runner`)
- `description` — one-line summary
- `instructions` — detailed workflows and guidelines in markdown

Promotion path: staged skill → accepted → active permanent skill (visible via
`list_skills`).

## Skill Authoring Guidance

1. Name it for what it *does*, not what file it touches.
2. Instructions must be self-contained — a fresh session reads only the SKILL.md.
3. Include failure modes and verification steps in the instructions.
4. One skill, one job. Chain skills externally rather than building mega-skills.
