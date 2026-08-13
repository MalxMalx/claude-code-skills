---
name: bootstrap-fastapi-ai-project
description: Bootstrap a new Python/FastAPI project set up for AI/LLM work (OpenAI API, async, Postgres via psycopg3) with a fully-configured dev experience — uv, ruff (lint+format), mypy/pyright strict typing, pytest, pre-commit hooks, VS Code format-on-save, a health-check endpoint, and a passing test for it. Use this whenever the user asks to "bootstrap a fastapi ai project", "scaffold a new fastapi/openai project", "set up a new python ai project like last time", or wants a fresh Python project with the same tooling (uv + ruff + mypy + pytest + pre-commit) as their reference project. Trigger this proactively whenever the user is starting a brand-new Python service intended to call an LLM API, even if they don't name all the tools explicitly.
---

# Bootstrap FastAPI AI Project

Recreates a known-good starting point for a Python + FastAPI + OpenAI-API project, with a TypeScript-like dev experience (format-on-save, live type checking, lint-on-commit) and a working health-check endpoint + test to prove the scaffold runs end to end.

This skill assumes `uv` and `git` are installed on the machine. If either is missing, tell the user to install it first (`curl -LsSf https://astral.sh/uv/install.sh | sh` for uv) rather than trying to work around it.

## Steps

1. **Ask for the project name** if not given (used as both the folder name and, with underscores, the importable package name). Ask where it should live if ambiguous.

2. **Scaffold the project**
   ```bash
   uv init <project-name> --app --no-package --python 3.12
   cd <project-name>
   rm main.py
   mkdir -p app tests
   touch app/__init__.py
   ```
   `--no-package` is required, not optional. As of uv 0.12, a bare `uv init` defaults to a **packaged** project: it generates a `src/<pkg>/` layout, a `[project.scripts]` entry, and a `[build-system]` using the `uv_build` backend — the exact shape this skill does not want. `--no-package` restores the flat layout with a top-level `main.py` and no `[build-system]`, which this skill then replaces with a hand-rolled `app/` package — the more common shape for real-world FastAPI services. (Older uv versions defaulted to flat and needed no flag; passing `--app --no-package` is correct and harmless on both.)

   Note `uv init` already runs `git init` for you, so a separate `git init` is redundant.

   Do not use `src/` layout here and do not add `--package` unless the user specifically says this project needs to be pip-installable (e.g. a shared internal library).

3. **Add dependencies**
   ```bash
   uv add fastapi uvicorn "openai" "psycopg[binary,pool]" alembic pydantic-settings
   uv add --dev ruff mypy pytest pytest-asyncio pre-commit httpx
   ```
   (Drop `psycopg`/`alembic` if the user says this particular project has no DB component — everything else is still relevant.)

4. **Copy in the config assets** from this skill's `assets/` directory into the new project, adapting as needed:
   - `assets/gitignore` → `.gitignore` (project root)
   - `assets/vscode_settings.json` → `.vscode/settings.json`
   - `assets/pre-commit-config.yaml` → `.pre-commit-config.yaml`
   - `assets/pyproject_tool_config.toml` → append its contents into the `pyproject.toml` that `uv init` created (do not overwrite the `[project]` section uv generated). This includes `pythonpath = ["."]` under `[tool.pytest.ini_options]` — required so `import app` resolves in tests without installing the project as a package.
   - `assets/main.py` → `app/main.py`
   - `assets/test_health.py` → `tests/test_health.py` (imports from `app.main` directly, no substitution needed)

5. **Install the git hook**
   ```bash
   uv run pre-commit install
   ```

6. **Verify the scaffold works** before handing back to the user:
   ```bash
   uv run pytest
   uv run ruff check .
   uv run mypy .
   ```
   All three should pass cleanly on the freshly generated code. If `mypy --strict` complains about anything in the generated `app/main.py`/`tests/test_health.py`, fix the template files in this skill's `assets/` directory too, so the issue doesn't recur next time the skill runs.

7. **Report back concisely**: project path, confirmation that health check + test pass, and remind the user to select the `.venv` interpreter in VS Code (Cmd/Ctrl+Shift+P → "Python: Select Interpreter") so Pylance picks up strict type checking. Run the server with `uv run uvicorn app.main:app --reload`.

## Notes

- This is deliberately framework-light: raw `openai` SDK, raw `psycopg3`, no ORM, no PydanticAI/LangChain. If the user wants one of those added, add it as an extra step, not a replacement for this base scaffold.
- `psycopg[binary]` is for local dev convenience (pre-built wheel, no compiler needed). Mention that production deploys typically prefer `psycopg[c]` built for the target system, but don't act on it unless asked.
- If the user wants a DB pool + Alembic migrations wired up too (not just installed), that's a natural follow-up but outside this skill's scope — treat it as a separate ask so this scaffold stays fast and minimal.
- No `src/` layout, no build backend, no editable install. This intentionally trades away the "tests exercise the installed package" guarantee for simplicity, which is the right trade for a service that's only ever run via `uvicorn`. If the user later wants to package this as an installable library, that's a distinct, deliberate migration — not something to default into.
