---
name: bootstrap-python-repo
description: Use when creating a new Python project from scratch, setting up a Python repo skeleton, or when the user says "bootstrap", "scaffold", or "init" a Python project. Also trigger when the user says "new python app", "start a python project", "set up a python package", "create a python CLI", or is in an empty directory and wants to start coding in Python. Covers uv, pytest, ruff, pyright, git hooks, CLAUDE.md, and src layout.
---

# Bootstrap Python Repo

Set up a new Python project with uv, ruff, pytest, pyright, conventional commits, CLAUDE.md, and src layout.

Assumes the user has already created and `cd`'d into the project directory.

## When to Use

- User asks to create/bootstrap/scaffold/init a new Python project
- User wants a repo skeleton with modern Python tooling
- User is in an empty directory and wants to start a Python project

## Setup Steps

### 1. Ask the user

Before generating files, ask:

- **Project name** (default: current directory name)
- **One-line project description** (used in CLAUDE.md and pyproject.toml)
- **Python version** (default: `>=3.12`)
- **License** (default: MIT)
- **Whether they need a `.env.example`** and if so, what variables

### 2. Initialize git and uv

The user is already inside the project directory. Initialize in place:

```bash
git init .
uv init --app --no-readme
```

Then delete the generated `hello.py` and replace `pyproject.toml` with the template in step 3:

```bash
rm hello.py
```

### 3. Create files

Generate these files, substituting `<project-name>` and `<project-description>` throughout:

#### `pyproject.toml`

```toml
[project]
name = "<project-name>"
version = "0.0.0"
description = "<project-description>"
requires-python = ">=3.12"
dependencies = []

[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"

[tool.hatch.build.targets.wheel]
packages = ["src/<project-name>"]

[dependency-groups]
dev = [
    "pytest>=8.0",
    "pyright>=1.1",
    "ruff>=0.9",
]

[tool.ruff]
line-length = 88

[tool.ruff.lint]
select = ["E", "F", "I", "UP"]

[tool.pytest.ini_options]
testpaths = ["tests"]
markers = [
    "integration: marks tests as integration tests (deselect with '-m \"not integration\"')",
]

[tool.pyright]
pythonVersion = "3.12"
typeCheckingMode = "standard"
venvPath = "."
venv = ".venv"
```

#### `.gitignore`

```
# Python
__pycache__/
*.pyc
*.pyo
*.pyd

# Virtual environments
.venv/
venv/

# Distribution
dist/
build/
*.egg-info/

# uv
.python-version

# Tools
.ruff_cache/
.pytest_cache/
.pyright/

# Secrets
.env
```

#### `src/<project-name>/__init__.py`

Empty file.

#### `tests/__init__.py`

Empty file.

#### `.githooks/commit-msg`

```bash
#!/usr/bin/env bash
set -euo pipefail

commit_msg_file="$1"
commit_msg=$(cat "$commit_msg_file")

# Skip merge commits
if echo "$commit_msg" | grep -qE "^Merge "; then
    exit 0
fi

pattern="^(feat|fix|docs|style|refactor|test|chore|perf|ci|build)(\(.+\))?: .+"

if ! echo "$commit_msg" | grep -qE "$pattern"; then
    echo ""
    echo "ERROR: Invalid commit message format."
    echo ""
    echo "Expected: <type>(<optional scope>): <description>"
    echo "Example:  feat(core): add initial implementation"
    echo ""
    echo "Valid types: feat, fix, docs, style, refactor, test, chore, perf, ci, build"
    echo ""
    echo "Your message: $commit_msg"
    echo ""
    echo "Use --no-verify to bypass (e.g. for WIP commits)."
    echo ""
    exit 1
fi
```

#### `.githooks/pre-commit`

```bash
#!/usr/bin/env bash
set -euo pipefail

# Check for staged Python files (newline-delimited, just for the guard)
staged_py=$(git diff --cached --name-only --diff-filter=ACM | grep '\.py$' || true)

# No-op cleanly when no Python files are staged
if [ -z "$staged_py" ]; then
    exit 0
fi

# Note: ruff checks working-tree files, not the staged snapshot.
# For a single-developer workflow this is acceptable.

# Use null-delimited pipeline (handles filenames with spaces).
# The brace group with || true prevents set -o pipefail from aborting on grep's
# non-zero exit if (unexpectedly) no .py files match — xargs -0 handles empty input cleanly.
{ git diff --cached --name-only --diff-filter=ACM -z | grep -z '\.py$' || true; } | xargs -0 uv run ruff check
{ git diff --cached --name-only --diff-filter=ACM -z | grep -z '\.py$' || true; } | xargs -0 uv run ruff format --check

echo "OK"
```

#### `CLAUDE.md`

```markdown
# <Project-Name> — Claude Code Reference

## Project Overview

<project-description>

---

## Architecture

- `src/<project-name>/` — production source code (src layout)
- `tests/` — mirrors `src/<project-name>/` structure (e.g. `src/<project-name>/foo.py` → `tests/test_foo.py`)
- `.githooks/` — git hooks, installed with `git config core.hooksPath .githooks`

---

## Coding Conventions

- **Type hints required** on all function signatures (parameters and return types). No bare `def`.
- **No `Any`** without an inline comment explaining why it cannot be avoided.
- **`ruff` is enforced** via the pre-commit hook. Rules: E, F, I (isort), UP (pyupgrade). Line length: 88.
- **No `print()`** in production code. Use the `logging` module.
- **No secrets in code.** API keys and credentials go in `.env` (gitignored).

---

## Testing

- **Runner:** `uv run pytest`
- **Location:** `tests/` — mirrors the `src/<project-name>/` structure.
- **Prefer unit tests.** Mock external API calls; don't make real HTTP requests in tests.
- **Integration tests** (real API calls, real network) must be marked:
  ```python
  @pytest.mark.integration
  ```
  Run only explicitly: `uv run pytest -m integration`
- Write the test first, then the implementation.

---

## Common Commands

| Task | Command |
|------|---------|
| Install deps | `uv sync` |
| Run tests | `uv run pytest` |
| Lint | `uv run ruff check .` |
| Format | `uv run ruff format .` |
| Type check | `uv run pyright` |
| Install hooks | `git config core.hooksPath .githooks` |

---

## Things to Avoid

- Broad `except Exception` blocks — catch specific exceptions.
- Mocking at levels that diverge from real API behaviour — test logic in isolation from I/O rather than deeply mocking internals.

---

## Self-Update Note

If you (Claude) introduce a new module, establish a new pattern, or make an architectural decision
that isn't reflected here, flag it and suggest a CLAUDE.md update. This document should stay current.
```

Adapt this template to the project: fill in the project name and description, and if the user mentioned specific architectural details or extra conventions during step 1, incorporate them. The template is a starting point, not a rigid form.

#### `.env.example` (if requested)

```
# <describe the variable>
VARIABLE_NAME=your_value_here
```

#### `LICENSE` (MIT, if selected)

Standard MIT license text with the current year and user's name.

### 4. Make hooks executable and install

```bash
chmod +x .githooks/commit-msg .githooks/pre-commit
git config core.hooksPath .githooks
```

### 5. Install dependencies

```bash
uv sync
```

### 6. Verify

```bash
uv run ruff check .
uv run pytest
```

### 7. Initial commit

Stage all files and commit:

```
chore: add project foundation
```

## Common Commands

After setup, these commands are available:

| Task | Command |
|------|---------|
| Install deps | `uv sync` |
| Add a dependency | `uv add <package>` |
| Add a dev dependency | `uv add --group dev <package>` |
| Run tests | `uv run pytest` |
| Lint | `uv run ruff check .` |
| Format | `uv run ruff format .` |
| Type check | `uv run pyright` |
| Re-install hooks | `git config core.hooksPath .githooks` |

## Directory Structure

```
<project-name>/
  .githooks/
    commit-msg
    pre-commit
  src/
    <project-name>/
      __init__.py
  tests/
    __init__.py
  .env.example      # if requested
  .gitignore
  CLAUDE.md
  LICENSE
  pyproject.toml
  uv.lock
```
