# The Complete Guide to UV (Python Package & Project Manager)

> A comprehensive, end-to-end guide to **UV**: what it is, installation, core concepts, project and dependency management, Python version management, comparison with other tools, and CI/CD usage.

---


## Table of Contents

- [What is UV?](#1-what-is-uv)
- [Why UV? (Advantages over legacy tools)](#2-why-uv-advantages-over-legacy-tools)
- [Installing UV](#3-installing-uv)
- [Core Concepts](#4-core-concepts)
- [Project Management with UV](#5-project-management-with-uv)
- [Dependency Management](#6-dependency-management)
- [Python Version Management](#7-python-version-management)
- [Virtual Environment Management](#8-virtual-environment-management)
- [Running Scripts and Tools](#9-running-scripts-and-tools)
- [Locking Dependencies (Lock File)](#10-locking-dependencies-lock-file)
- [Test Scenarios & Benchmarking UV vs pip](#11-test-scenarios--benchmarking-uv-vs-pip)
- [UV vs Other Tools — Comparison](#12-uv-vs-other-tools--comparison)
- [Command Cheat Sheet](#13-command-cheat-sheet)
- [Using UV in CI/CD](#14-using-uv-in-cicd)
- [Project Setup Checklist](#15-project-setup-checklist)
- [Further Resources](#16-further-resources)

---

## 1. What is UV?

**UV** is a Python package and project manager built by **Astral** — the same company behind the popular linter/formatter `Ruff`. UV is written in **Rust** and aims to replace a fragmented set of Python tooling (`pip`, `pip-tools`, `pipx`, `poetry`, `virtualenv`, `pyenv`) with a single, fast, unified tool.

The headline claim: UV can be **10–100x faster** than `pip` for common operations such as installing packages, creating virtual environments, and resolving dependencies — thanks to its Rust core, parallel downloads, and a global cache.

---

## 2. Why UV? (Advantages over legacy tools)

- **Extreme speed**: Dependency resolution and installation are dramatically faster due to the Rust implementation and parallelized network I/O.
- **All-in-one tooling**: Replaces the need to juggle `pyenv` + `virtualenv` + `pip` + `pip-tools` + `pipx` separately.
- **Automatic Python version management**: No need to manually install Python versions — UV downloads and manages them for you.
- **Reliable lock file**: `uv.lock` pins exact versions of every dependency (direct and transitive) for reproducible builds.
- **Standards-compliant**: Built around the official `pyproject.toml` standard (PEP 621) — no proprietary config formats.
- **Global cache with hard links**: Packages are downloaded once and shared across projects via hard links, saving both time and disk space.
- **Drop-in `pip` compatibility mode**: `uv pip install` works as a fast drop-in replacement inside existing pip-based workflows.

---

## 3. Installing UV

### Linux and macOS

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

### Windows (PowerShell)

```powershell
powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"
```

### Via pip (if Python is already installed)

```bash
pip install uv
```

### Via pipx

```bash
pipx install uv
```

### Via Homebrew (macOS)

```bash
brew install uv
```

### Verify installation

```bash
uv --version
```

---

## 4. Core Concepts

| Concept | Description |
|---|---|
| `pyproject.toml` | The main project definition file — metadata and dependencies (official Python standard) |
| `uv.lock` | Lock file that pins the exact resolved version of every dependency (direct + transitive) |
| `.venv` | The project's virtual environment, created automatically by UV inside the project folder |
| Global Cache | A shared, system-wide package cache reused across all projects on the machine |
| Resolver | UV's dependency resolution engine that finds compatible versions for all packages |

---

## 5. Project Management with UV

### Create a new project

```bash
uv init my-project
cd my-project
```

### Initialize a project in the current directory

```bash
uv init
```

### Typical project structure UV creates

```
my-project/
├── .venv/
├── .python-version
├── pyproject.toml
├── uv.lock
├── README.md
└── src/
    └── my_project/
        └── __init__.py
```

---

## 6. Dependency Management

### Add a package

```bash
uv add requests
```

### Add with a version constraint

```bash
uv add "requests>=2.31,<3.0"
```

### Add a development-only dependency

```bash
uv add --dev pytest ruff
```

### Add an optional dependency group

```bash
uv add --optional docs mkdocs
```

### Remove a package

```bash
uv remove requests
```

### Install all project dependencies (per pyproject.toml)

```bash
uv sync
```

### Upgrade a single package

```bash
uv lock --upgrade-package requests
```

### Upgrade all packages

```bash
uv lock --upgrade
```

---

## 7. Python Version Management

UV can download and manage multiple Python versions itself — no `pyenv` required.

### Install a specific Python version

```bash
uv python install 3.12
```

### List installed Python versions

```bash
uv python list
```

### Pin a Python version for a project

```bash
uv python pin 3.12
```

This creates a `.python-version` file that locks the interpreter version for the project.

### Install multiple versions at once

```bash
uv python install 3.10 3.11 3.12
```

---

## 8. Virtual Environment Management

### Create a virtual environment

```bash
uv venv
```

### Create with a specific Python version

```bash
uv venv --python 3.11
```

### Activate the virtual environment

```bash
# Linux/macOS
source .venv/bin/activate

# Windows
.venv\Scripts\activate
```

> Note: For most UV commands (like `uv run`), manual activation isn't necessary — UV automatically detects and uses the correct environment.

---

## 9. Running Scripts and Tools

### Run a Python script inside the project environment

```bash
uv run main.py
```

### Run a command (e.g. tests)

```bash
uv run pytest
```

### Run a tool temporarily without installing it (like `npx`)

```bash
uvx ruff check .
```

`uvx` is shorthand for `uv tool run` — it runs a tool in an isolated, ephemeral environment without polluting your project.

### Install a tool globally (persistent)

```bash
uv tool install ruff
```

### Run a standalone script with inline dependencies (PEP 723)

```bash
uv run --with requests script.py
```

Or declare dependencies directly inside the script using inline metadata:

```python
# /// script
# dependencies = ["requests"]
# ///

import requests
print(requests.get("https://example.com").status_code)
```

---

## 10. Locking Dependencies (Lock File)

The `uv.lock` file records the exact resolved version of every dependency (direct and transitive), guaranteeing reproducible installs across machines and environments.

### Create or update the lock file

```bash
uv lock
```

### Install strictly according to the lock file (no version drift)

```bash
uv sync --locked
```

### Check if the lock file is out of date (useful in CI)

```bash
uv lock --check
```

> `uv.lock` should always be committed to version control so every team member and CI server installs the exact same dependency versions.

---

## 11. Test Scenarios & Benchmarking UV vs pip

To evaluate UV's real-world impact on your workflow, it helps to benchmark it against `pip` under a few common scenarios and document the results.

### Suggested test scenarios

| Scenario | Purpose |
|---|---|
| Fresh virtual environment creation | Compare `uv venv` vs `python -m venv` |
| Installing a small project (~10 deps) | Baseline install speed |
| Installing a large project (100+ deps) | Stress-test the resolver and cache |
| Re-installing with warm cache | Measure cache hit performance |
| Installing on a CI runner (cold cache) | Real-world CI speed impact |

### Simple timing benchmark

```bash
# pip baseline
time python -m venv .venv-pip && source .venv-pip/bin/activate && time pip install -r requirements.txt

# uv comparison
time uv venv .venv-uv && time uv pip install -r requirements.txt
```

### Result logging template

| Date | Scenario | Tool | Dependencies | Cold Cache (s) | Warm Cache (s) | Notes |
|---|---|---|---|---|---|---|
| 2026-06-01 | Small project | pip | 10 | 8.2 | 6.5 | - |
| 2026-06-01 | Small project | uv | 10 | 0.6 | 0.05 | ~14x faster cold |
| 2026-06-01 | Large project | pip | 120 | 95.0 | 70.0 | - |
| 2026-06-01 | Large project | uv | 120 | 4.1 | 0.3 | ~23x faster cold |

> Actual speedups vary by network, package sizes, and whether native extensions need compilation — always benchmark against your own project.

---

## 12. UV vs Other Tools — Comparison

| Tool | Replaced by UV? | Notes |
|---|---|---|
| `pip` | ✅ | Package installation, but much faster |
| `pip-tools` | ✅ | Dependency locking (`uv lock`) |
| `virtualenv` / `venv` | ✅ | Virtual environment creation (`uv venv`) |
| `pyenv` | ✅ | Python version management (`uv python`) |
| `pipx` | ✅ | Isolated tool execution (`uvx`) |
| `poetry` | Largely ✅ | Project & dependency management, with much higher speed |
| `conda` | ❌ (partial) | UV only manages Python packages, not system-level/non-Python packages |

---

## 13. Command Cheat Sheet

```bash
uv init                     # Create a new project
uv add <package>            # Add a dependency
uv remove <package>         # Remove a dependency
uv sync                     # Install all dependencies per pyproject.toml
uv lock                     # Update uv.lock
uv lock --check             # Verify lock file is up to date
uv run <command>            # Run a command inside the project environment
uv venv                     # Create a virtual environment
uv python install <ver>     # Install a Python version
uv python pin <ver>         # Pin a Python version for the project
uvx <tool>                  # Run a tool temporarily (ephemeral)
uv tool install <tool>      # Install a tool globally
uv pip install <package>    # pip-compatible mode (if needed)
uv cache clean              # Clear the global cache
```

---

## 14. Using UV in CI/CD

Example GitHub Actions workflow using UV:

```yaml
name: CI

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Install uv
        uses: astral-sh/setup-uv@v3

      - name: Install dependencies
        run: uv sync --locked

      - name: Run tests
        run: uv run pytest

      - name: Lint
        run: uv run ruff check .
```

Key benefit in CI: thanks to caching and parallel installs, pipeline execution time drops significantly, especially on repeated runs where the cache is warm.

### Caching UV in GitHub Actions

```yaml
      - name: Install uv
        uses: astral-sh/setup-uv@v3
        with:
          enable-cache: true
```

---

## 15. Project Setup Checklist

- [ ] Is UV installed? (`uv --version`)
- [ ] Was the project created with `uv init`?
- [ ] Is the Python version pinned with `uv python pin`?
- [ ] Are core dependencies added via `uv add`?
- [ ] Are dev dependencies (tests, linters) separated from core dependencies?
- [ ] Is `uv.lock` committed to version control?
- [ ] Is `.venv/` listed in `.gitignore`?
- [ ] Does CI/CD use `uv sync --locked`?
- [ ] Is the global cache enabled in CI for faster builds?

---

## 16. Further Resources

- [UV Official Documentation](https://docs.astral.sh/uv/)
- [UV GitHub Repository](https://github.com/astral-sh/uv)
- [Astral Blog](https://astral.sh/blog)
- [PEP 621 — Storing project metadata in pyproject.toml](https://peps.python.org/pep-0621/)
- [PEP 723 — Inline script metadata](https://peps.python.org/pep-0723/)

---

<p align="center">
Prepared for team documentation and technical onboarding
</p>

