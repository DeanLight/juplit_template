# juplit_template

A [cookiecutter](https://cookiecutter.readthedocs.io/) template for starting a new juplit project.

## Installation

Install [Cookiecutter](https://cookiecutter.readthedocs.io/) so the `cookiecutter` command is available. Pick one method:

**uv** (matches the rest of this workflow):

```bash
uv tool install cookiecutter
```

**pip** (any Python environment):

```bash
pip install cookiecutter
```

**pipx** (isolated CLI install):

```bash
pipx install cookiecutter
```

**Homebrew** (macOS / Linux):

```bash
brew install cookiecutter
```

## Usage

```bash
cookiecutter gh:DeanLight/juplit_template
```

Or from a local clone:

```bash
cookiecutter path/to/juplit_template/
```

### Template variables

| Variable | Default | Description |
|---|---|---|
| `project_name` | My Juplit Project | Human-readable project name |
| `project_slug` | my_juplit_project | Package name (used in pyproject.toml) |
| `module_name` | my_juplit_project | Python module directory name |
| `author` | Your Name | Package author |
| `python_version` | >=3.12 | Python version constraint |
| `juplit_version` | >=0.1.0 | juplit version constraint |

## After generation

```bash
cd your_project_slug/
uv sync              # install dependencies
poe init             # install git hooks
poe sync             # generate .ipynb files from .py sources
```

## Workflow

- Edit `.py` files (jupytext percent format) — these are your source of truth
- Run `poe sync` to sync changes into `.ipynb` notebooks
- Run `poe clean` to remove all `.ipynb` files (useful before AI agent work)
- `.ipynb` files are gitignored by default
