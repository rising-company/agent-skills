# Agent Skills

Rising Company's collection of skills for AI coding agents. Skills are packaged
instructions and scripts that extend agent capabilities.

Skills follow the [Agent Skills](https://agentskills.io/) format.

## Available Skills

_None yet — this repo is the starting point. New skills will be added under
[`skills/`](./skills) as we build them._

## Skill Structure

Each skill lives in its own directory under `skills/` and contains:

- `SKILL.md` — instructions for the agent (required)
- `scripts/` — helper scripts for automation (optional)
- `references/` — supporting documentation read on demand (optional)

See [AGENTS.md](./AGENTS.md) for the full authoring guide.

## Installation

### Claude Code

Copy a skill directory into your user-level skills folder:

```bash
cp -r skills/<skill-name> ~/.claude/skills/
```

Or symlink during development so edits take effect immediately:

```bash
ln -s "$(pwd)/skills/<skill-name>" ~/.claude/skills/<skill-name>
```

### claude.ai

Add the skill to project knowledge, or paste the contents of `SKILL.md` into
the conversation.

## Contributing

1. Create a new directory under `skills/` using `kebab-case`.
2. Add a `SKILL.md` following the format in [AGENTS.md](./AGENTS.md).
3. Keep `SKILL.md` under 500 lines — put detail in `references/`.
4. Open a PR.

## License

Internal to Rising Company unless noted otherwise on a per-skill basis.
