# AGENTS.md

This file provides guidance to AI coding agents (Claude Code, Cursor, Copilot,
etc.) when working with code in this repository.

## Repository Overview

A collection of skills authored by Rising Company for use with Claude Code and
claude.ai. Skills are packaged instructions and scripts that extend an agent's
capabilities for specific tasks.

## Creating a New Skill

### Directory Structure

```
skills/
  {skill-name}/           # kebab-case directory name
    SKILL.md              # Required: skill definition
    scripts/              # Optional: executable helper scripts
      {script-name}.sh
    references/           # Optional: supporting docs read on demand
      {topic}.md
```

### Naming Conventions

- **Skill directory**: `kebab-case` (e.g., `deploy-to-staging`, `log-monitor`)
- **SKILL.md**: always uppercase, always this exact filename
- **Scripts**: `kebab-case.sh` (e.g., `deploy.sh`, `fetch-logs.sh`)

### SKILL.md Format

````markdown
---
name: {skill-name}
description: {One sentence describing when to use this skill. Include concrete trigger phrases the user would say, e.g. "Deploy my app", "Check logs".}
---

# {Skill Title}

{Brief description of what the skill does.}

## How It Works

{Numbered list explaining the skill's workflow.}

## Usage

```bash
bash ${CLAUDE_SKILL_DIR}/scripts/{script}.sh [args]
```

**Arguments:**

- `arg1` — description (defaults to X)

**Examples:**

{Two or three common usage patterns.}

## Output

{Show example output the user will see.}

## Present Results to User

{Template for how the agent should format results back to the user.}

## Troubleshooting

{Common issues and solutions — especially network or permissions errors.}
````

### Best Practices for Context Efficiency

Skills are loaded on demand — only the skill `name` and `description` are loaded
at startup. The full `SKILL.md` loads into context only when the agent decides
the skill is relevant. To minimize context usage:

- **Keep `SKILL.md` under 500 lines** — put detailed reference material in
  separate files under `references/`.
- **Write specific descriptions** — helps the agent know exactly when to
  activate the skill.
- **Use progressive disclosure** — reference supporting files that are read
  only when needed.
- **Prefer scripts over inline code** — script execution does not consume
  context (only the output does).
- **File references work one level deep** — link directly from `SKILL.md` to
  supporting files.

### Script Requirements

- Use `#!/bin/bash` shebang.
- Use `set -euo pipefail` for fail-fast behavior.
- Write status messages to stderr: `echo "Message" >&2`.
- Write machine-readable output (JSON) to stdout.
- Include a cleanup `trap` for any temp files.
- Reference scripts via the skill's directory — do not hard-code absolute paths
  outside the skill.

### Description Quality

The `description` field is the single most important line in a skill. It is the
only thing the agent sees when deciding whether to load the skill. A good
description:

- Names the concrete task ("Deploy a Next.js app to our staging env" — not
  "deployment helper").
- Lists trigger phrases the user is likely to say.
- States what the skill is **not** for, if there's an obvious adjacent skill it
  could be confused with.

### Testing a Skill Locally

Symlink the skill into your local Claude Code skills directory so edits take
effect immediately:

```bash
ln -s "$(pwd)/skills/{skill-name}" ~/.claude/skills/{skill-name}
```

Then invoke it from a Claude Code session and iterate.

## When Helping Users in This Repo

- **This repo is PUBLIC.** Keep every skill generic and portfolio-agnostic — no
  internal app/repo names, private paths, branch names, infra details, or
  security/roadmap specifics. Push that context to private memory or a private
  repo and reference it only abstractly. (Note: a leaked commit can persist in a
  merged PR's `refs/pull/N/head` even after a history rewrite — get it right before merging.)
- If the user asks you to create a new skill, scaffold the directory under
  `skills/{skill-name}/` with a `SKILL.md` matching the format above.
- Do not invent fake examples in `README.md` — only list skills that actually
  exist in `skills/`.
- Keep `SKILL.md` files under 500 lines; refactor detail into `references/`
  before exceeding that.
