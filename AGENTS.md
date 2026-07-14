# Repository Guidelines

## Project Structure & Module Organization

Core runtime code lives in `pico/`. Provider adapters are under `pico/providers/`, evaluation utilities under `pico/evaluation/`, and optional capabilities under `pico/features/`. The CLI entry points are `pico/cli.py` and `pico/__main__.py`. Tests are in `tests/`, with supporting data in `tests/fixtures/`. Keep benchmark definitions and results in `benchmarks/`, experiment utilities in `scripts/`, architecture notes in `docs/`, and screenshots in `assets/screenshots/`. `examples/mini-pico/` is a standalone reduced implementation with its own package and tests.

## Build, Test, and Development Commands

- `pip install -e .` installs the `pico` command in editable mode.
- `uv run pico --cwd /path/to/repo` starts the interactive agent against a workspace.
- `uv run pico "inspect the test failures"` runs a one-shot task.
- `python -m pico` invokes the module entry point directly.
- `uv run pytest tests -q` runs the main test suite.
- `uv run ruff check pico tests scripts` checks source, tests, and utilities for lint violations.
- `uv run pytest examples/mini-pico/tests -q` validates the standalone example.

## Coding Style & Naming Conventions

Use four-space indentation and standard Python conventions: `snake_case` for functions, variables, and modules; `PascalCase` for classes; and `UPPER_SNAKE_CASE` for constants. Add type hints where they clarify public interfaces or non-obvious data flow. Keep provider-specific behavior inside `pico/providers/` and avoid adding runtime dependencies without a clear need. Run Ruff before submitting changes.

## Testing Guidelines

Tests use pytest and follow `test_*.py` file names with `test_*` functions. Place focused unit tests beside the closest existing test module, and add fixtures under `tests/fixtures/` only when inline setup would obscure intent. Cover success, failure, and security-sensitive paths for changes to tools, persistence, provider selection, or command execution. No numeric coverage threshold is configured; regression behavior should still be explicitly tested.

## Commit & Pull Request Guidelines

Recent history uses Conventional Commit prefixes such as `feat:`, `fix:`, `docs:`, and `refactor:`. Keep each commit scoped to one coherent change and write the subject in imperative form. Pull requests should explain the motivation and behavior change, list verification commands, link relevant issues, and call out configuration or compatibility effects. Include screenshots when CLI output or documented UI changes.

## Security & Configuration

Copy `.env.example` to `.env` for local provider settings. Never commit API keys, tokens, `.env`, or generated `.pico/` session and run artifacts. Ensure new secret environment variables are included in redaction behavior and relevant security tests.
