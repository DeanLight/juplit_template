# Design — template support for artifact notebooks

- **Task:** [Spec: \[juplit\] Artifact notebooks — committed outputs and the agent execution loop](https://app.notion.com/p/3c30dbff56478187a081ef2a1c00eba2) (TASK-96, P1)
- **Spec:** [\[juplit\] Artifact notebooks …](https://app.notion.com/p/3c30dbff56478130b1cac805161c95cb) — *"TODO this will require editing the juplit template repo as well."*
- **Branch:** `claude/artifact-notebooks-execution-0xt0im`
- **Depends on:** the juplit PR on the same branch name in [DeanLight/juplit](https://github.com/DeanLight/juplit) — this template is unusable until that release is on PyPI, so this PR merges **after** it and pins the version that ships the feature.

This is the companion half of one spec. It ships no logic: a cookiecutter template is
configuration, and everything here is a key, a hook, a task, or a paragraph.

---

## 0. Justify existence

**Does the template need to change at all?** Yes, for two reasons that are not the same reason.

1. **The guard has to be wired, or it does not run.** `juplit check` is a pre-commit / CI guard.
   A generated project that does not install the hook and does not run the step gets the failure
   mode the spec exists to prevent — outputs that contradict their code, committed, unnoticed.
   The template is the only place this can be wired once for every future project.
2. **Discoverability is the stated point of the wrapper.** From the spec: *"so that the agents see
   this as an ability when they look at juplit commands in the pyproject yaml for projects."* An
   agent reads `[tool.poe.tasks]` to learn what a repo can do. `juplit html` and `juplit check`
   have to appear in that table or, for the agent's purposes, they do not exist.

Component by component:

- **`artifact_notebooks` key in the generated `pyproject.toml`**
  - *Needs to exist:* yes — an empty, commented key is how a user discovers the feature exists at all.
  - *Already solved:* no. *Smallest form:* one commented line.
- **`poe check`**
  - *Needs to exist:* yes — reason 1 above.
  - *Already solved:* no. *Smallest form:* one `cmd` task.
- **`poe html`**
  - *Needs to exist:* yes — reason 2 above.
  - *Already solved:* no. *Smallest form:* one `cmd` task.
- **`juplit check` pre-commit hook**
  - *Needs to exist:* yes — reason 1; the template already installs `juplit sync` the same way.
  - *Already solved:* no. *Smallest form:* four lines beside the existing hook.
- **`juplit check` CI step**
  - *Needs to exist:* yes — the hook only protects committers, not the merge.
  - *Already solved:* no. *Smallest form:* one `run:` line.
- **`.juplit/` in `.gitignore`**
  - *Needs to exist:* yes — kernel sessions, logs, sockets and the `last-run` cache are local state.
  - *Already solved:* no. *Smallest form:* one line.
- **Un-ignore comment in `.gitignore`**
  - *Needs to exist:* yes — issue #3: preserved-but-never-committed is indistinguishable from the feature not working.
  - *Already solved:* no. *Smallest form:* two comment lines.
- **`notebook_src_dir` → `notebook_src_dirs`**
  - *Needs to exist:* yes, incidentally — the template still emits the legacy singular key, so `docs/` is not scanned at all in a generated project.
  - *Already solved:* no. *Smallest form:* one line.
- **`juplit_version` default bump**
  - *Needs to exist:* yes — the generated project must not resolve to a juplit without these commands.
  - *Already solved:* no. *Smallest form:* one JSON value.

**Cut outright:**

- **An example `experiments/` notebook** — spec out of scope: *"a template is a file you copy"*. An example artifact notebook would also have to ship committed outputs, which cookiecutter cannot produce.
- **A `poe kernel` / `poe try` task** — they take a notebook path or a snippet; they are agent commands, used as `juplit …` directly. Putting them in the poe table would imply a repo-wide default that does not exist.

---

## 1. Pseudocode

None — no code is added. The complete change is the file-by-file diff in §3.

---

## 2. Libraries and dependencies

**New dependencies: none.**

- `juplit` is already the template's dev dependency; only its version floor moves.
- `nbconvert`, which `poe html` shells out to, already arrives with `mkdocs-jupyter` in the
  template's dev group — verified in the juplit repo, whose dev group is the same shape
  (`nbconvert 7.17.1` present, no direct declaration).
- `jupyter_client` arrives transitively with the new juplit release; the template declares nothing.

---

## 3. File locations

**`cookiecutter.json`** — MODIFIED: `juplit_version` default `">=0.0.1"` → the release that ships
artifact notebooks (`">=0.1.0"`, filled in when the juplit PR is tagged).

**`{{cookiecutter.project_slug}}/pyproject.toml`** — MODIFIED, ~8 lines:

```toml
[tool.poe.tasks]
# … existing init / sync / nb / clean / test …
check = {cmd = "juplit check", help = "Fail if a committed notebook's outputs contradict its .py"}
html  = {cmd = "juplit html",  help = "Render a notebook to standalone HTML (nbconvert)"}

[tool.juplit]
notebook_src_dirs = ["{{cookiecutter.module_name}}", "docs"]
# Pairs whose .ipynb is committed WITH its outputs, because the outputs are the
# deliverable (experiment notebooks). Their outputs survive `nb`/`sync`/`clean`, and
# `juplit check` fails if they stop matching the .py that produced them.
# Remember to un-ignore the .ipynb in .gitignore — see the comment there.
artifact_notebooks = []   # e.g. ["experiments/**/*.py"]
```

**`{{cookiecutter.project_slug}}/.pre-commit-config.yaml`** — MODIFIED, +6 lines: a second local
hook beside `juplit-sync`, `id: juplit-check`, `entry: juplit check`, `pass_filenames: false`,
`always_run: true`. Ordered after sync, so the check sees the synced state.

**`{{cookiecutter.project_slug}}/.gitignore`** — MODIFIED, +4 lines: `.juplit/` (kernel sessions,
logs, sockets, `last-run.json`), and above the existing `*.ipynb` rules:

```gitignore
# Artifact notebooks are committed WITH their outputs — un-ignore each one you declare
# in [tool.juplit] artifact_notebooks, e.g.:
# !experiments/ablation.ipynb
```

**`{{cookiecutter.project_slug}}/.github/workflows/ci.yml`** — MODIFIED, +1 step after `pytest`:
`- run: uv run juplit check`. It is a no-op (exit 0, one line of output) until the project
declares an artifact, so it costs nothing for projects that never opt in.

**`{{cookiecutter.project_slug}}/README.md`** — MODIFIED, ~25 lines: two rows in the command
table (`poe check`, `poe html`) and a short **Artifact notebooks** section — what they are, the
one key, the un-ignore line, and a pointer to the juplit tutorial page for the agent loop. No
duplication of the tutorial itself; the template README is a card, not a manual.

**`README.md`** (the template repo's own) — MODIFIED, +3 lines: the `juplit_version` row in the
variables table, and one line under "Workflow" noting that `poe check` guards committed outputs.

---

## 4. Testing outline

The template has no test suite (it is cookiecutter templating), so the check is a generation
smoke run, done by hand before requesting review and recorded in the PR body:

- happy: `cookiecutter --no-input path/to/juplit_template/` generates without a Jinja error, and
  the generated `pyproject.toml` parses with `tomllib` — boundary: `artifact_notebooks = []` is
  present and empty.
- happy: `uv sync --all-groups && uv run poe check` in the generated project exits 0 and reports
  no artifacts configured.
- happy: `uv run pre-commit run --all-files` passes with both hooks installed.
- error-path: declare `artifact_notebooks = ["{{module}}/example.py"]`, execute the paired
  notebook, edit the `.py`, and assert `poe check` exits 1 naming the stale cell — i.e. the wiring
  actually reaches juplit's guard, which is the only thing this PR can get wrong.
- happy: `uv run poe html {{module}}/example.py` writes an `.html`.

---

## 5. Estimated scope

~7 files modified, **~70 lines added, ~10 modified**, zero added logic. Nothing here is
deletable without losing one of the two reasons in §0 — except the `notebook_src_dir` →
`notebook_src_dirs` fix, which is an unrelated (but one-line, and free) correction; say the word
and it moves to its own PR.
