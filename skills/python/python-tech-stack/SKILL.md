# Python Tech Stack

Set up the Python toolchain and project with:

- uv for Python env + dependency management + lockfile + task runner
- Python 3.12+ for runtime
- Ruff for linting + formatting
- basedpyright for typechecking (strict)
- pytest (+ `pytest-cov`) for unit tests + coverage

Use the latest stable Python toolchain via `uv add ...` / `uv add --dev ...`. Do not manually pin versions in `pyproject.toml` unless required.
