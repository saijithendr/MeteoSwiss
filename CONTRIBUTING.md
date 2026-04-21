# Contributing to MeteoSwiss

Thank you for considering a contribution! This guide covers everything you need to get started.

## Table of Contents
- [Code of Conduct](#code-of-conduct)
- [Getting Started](#getting-started)
- [Development Workflow](#development-workflow)
- [Running Tests](#running-tests)
- [Code Style](#code-style)
- [Submitting a Pull Request](#submitting-a-pull-request)
- [Reporting Bugs](#reporting-bugs)

---

## Code of Conduct

Please be respectful and constructive in all interactions. We follow the [Contributor Covenant](https://www.contributor-covenant.org/).

---

## Getting Started

### 1. Fork and clone

```bash
# Fork the repository on GitHub, then:
git clone https://github.com/<your-username>/MeteoSwiss.git
cd MeteoSwiss
```

### 2. Set up a virtual environment

```bash
python -m venv .venv
source .venv/bin/activate        # Linux / macOS
# .venv\Scripts\activate         # Windows
```

### 3. Install in editable mode with dev dependencies

```bash
pip install -e ".[dev]"
```

---

## Development Workflow

### Create a branch

Always branch from `master`:

```bash
git checkout master
git pull origin master
git checkout -b feature/your-feature-name   # new feature
# or
git checkout -b fix/short-bug-description   # bug fix
```

Branch naming conventions:
| Prefix | Use for |
|--------|---------|
| `feature/` | New functionality |
| `fix/` | Bug fixes |
| `docs/` | Documentation-only changes |
| `refactor/` | Code improvements without behavior change |

### Make your changes

- Keep each commit focused on a single logical change.
- Write clear commit messages: `fix: handle missing station data gracefully`.

---

## Running Tests

```bash
# Run the full test suite
pytest tests/ -v

# Run with coverage report
pytest tests/ --cov=src/swisswx --cov-report=term-missing

# Run only fast (non-integration) tests
pytest tests/ -m "not integration"

# Run a specific test file
pytest tests/test_meteo_swiss.py -v
```

### Using tox (tests across all supported Python versions)

```bash
tox            # all environments
tox -e py311   # single version
tox -e lint    # linting only
```

---

## Code Style

This project uses [Ruff](https://github.com/astral-sh/ruff) for both linting and formatting.

```bash
# Check for linting issues
ruff check src/ tests/

# Auto-fix fixable issues
ruff check --fix src/ tests/

# Format code
ruff format src/ tests/

# Check formatting without modifying files
ruff format --check src/ tests/
```

CI will reject PRs that fail linting or formatting checks.

---

## Submitting a Pull Request

1. Ensure all tests pass and linting is clean.
2. Push your branch to your fork:
   ```bash
   git push origin feature/your-feature-name
   ```
3. Open a Pull Request against `master` on the main repository.
4. Fill in the PR template — describe what changed, why, and how it was tested.
5. A maintainer will review your PR. Address any feedback and push additional commits.

### Branch protection on `master`

The `master` branch requires:
- At least **1 approving review** before merging.
- All **CI checks must pass** (lint, tests on Python 3.8–3.13, type-check).
- No direct pushes — all changes go through PRs.

---

## Reporting Bugs

Open an issue using the [Bug Report template](https://github.com/saijithendr/MeteoSwiss/issues/new?template=bug_report.md).

Please include:
- A clear description of the problem.
- A minimal code snippet that reproduces the issue.
- Your environment (Python version, OS, package versions).
