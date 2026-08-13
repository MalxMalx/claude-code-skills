# Claude Code personal skills

Personal skills for Claude Code. This repo *is* `~/.claude/skills/`, so cloning it
into place installs every skill it contains.

Claude Code does not sync this directory through your Claude account — it is plain
files on each machine. This repo is what keeps machines in sync.

## Set up on a new machine

`~/.claude/skills` must not already exist (or must be empty):

```bash
git clone git@github.com:MalxMalx/claude-code-skills.git ~/.claude/skills
```

Then restart Claude Code / Claude Desktop so it rescans for skills.

If the directory already exists with content you want to keep, clone elsewhere and
copy the skill folders in by hand instead.

## Staying in sync

```bash
git -C ~/.claude/skills pull
```

After editing a skill, commit and push from whichever machine you edited on, then
pull on the others. Restart Claude Code to pick up changes.

## Layout

One directory per skill, each with a `SKILL.md` whose YAML `name:` field matches its
directory name:

```
bootstrap-fastapi-ai-project/
├── SKILL.md
└── assets/
```

## Skills

- **bootstrap-fastapi-ai-project** — scaffolds a Python/FastAPI project for AI/LLM work
  (uv, ruff, mypy strict, pytest, pre-commit, VS Code format-on-save, health endpoint + test).

  Locally patched relative to the originally distributed `.skill`:
  - `uv init` now passes `--app --no-package`. As of uv 0.12 a bare `uv init` defaults to a
    packaged `src/` layout with a `[build-system]`, which is not the shape this skill wants.
  - `assets/gitignore` ignores `.venv/`, which it previously did not.
