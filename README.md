# mlops_practitioner_course

MLOps course work and Mini Project implementations.

The repository contains the course session materials inside separate `session_*`
folders, while the Mini Project implementation is built at the repository root
using a production-style Python package under `src/prodml`.

## Structure

```text
mlops_practitioner_course/
├── data/                  # Local datasets (not committed)
├── models/                # Saved model artifacts
├── notebooks/             # Baseline notebooks
├── reports/               # Module reports
├── src/
│   └── prodml/            # Production Python package
├── tests/                 # Mini Project tests
├── session_1/             # Course session materials
├── session_2/
├── session_3/
├── session_4/
├── pyproject.toml
├── .pre-commit-config.yaml
└── README.md
```

Each `session_*` folder contains the materials and examples for its corresponding
course session.

The Mini Project code is developed at the repository root and progressively
refactored into the `prodml` package.

## Module 1 Workflow

### Install

Install the package and development dependencies in editable mode:

```bash
pip install -e ".[dev]"
```

Alternative using `uv`:

```bash
uv pip install -e ".[dev]"
```

Editable mode means changes made inside `src/prodml/` are immediately available
without reinstalling the package after every edit.

### Lint

Run Ruff and Black:

```bash
ruff check src tests && black --check src tests
```

Ruff checks the code for linting issues and common mistakes.

Black checks that the Python files follow consistent formatting.

### Test

Run the project test suite and display coverage:

```bash
pytest -v --cov=src/prodml --cov-report=term-missing
```

The full Mini Project test suite is added later in Module 1.

### Train

Train the ride-duration model:

```bash
python -m prodml.train
```

The same training workflow can also be started through the installed command:

```bash
prodml-train
```

The training process:

```text
load Parquet data
        ↓
prepare and clean features
        ↓
split train / validation
        ↓
fit DictVectorizer
        ↓
train LinearRegression
        ↓
evaluate MAE and RMSE
        ↓
save models/model.pkl
```

The refactored package currently reproduces the original notebook baseline:

```text
Validation MAE:  4.22 minutes
Validation RMSE: 6.51 minutes
```

### Serve

The FastAPI service will be started with:

```bash
uvicorn prodml.api.main:app --reload --port 8000
```

The API implementation is added later in Module 1.

## Pre-commit Hooks

The repository uses [pre-commit](https://pre-commit.com) to run automated checks
before a commit and, for selected hooks, before a push.

The configuration lives in:

```text
.pre-commit-config.yaml
```

at the Git repository root.

If a hook detects a problem or modifies a file, the commit is stopped so the
changes can be reviewed and staged again before committing.

### Enable It

Install the hooks once per clone:

```bash
pre-commit install
```

To also install the pre-push hooks:

```bash
pre-commit install --hook-type pre-push
```

The `pre-commit` package itself is included in the development dependencies and
is installed with:

```bash
pip install -e ".[dev]"
```

or:

```bash
uv pip install -e ".[dev]"
```

### Run Hooks Manually

Run hooks against staged files automatically during:

```bash
git commit
```

Hooks can also be run manually on selected files:

```bash
pre-commit run --files \
    pyproject.toml \
    src/prodml/__init__.py \
    src/prodml/config.py \
    src/prodml/data.py \
    src/prodml/features.py \
    src/prodml/train.py \
    src/prodml/predict.py \
    src/prodml/api/__init__.py
```

Because this repository also contains older course-session files, running
`pre-commit run --all-files` may check or modify files outside the Mini Project.

## What the Hooks Do

There are two main categories of checks.

Fast static checks run before commits, while slower tests can run before pushes.

### Ruff

Ruff is used as the Python linter.

| Hook | Purpose |
|---|---|
| `ruff` (`--fix`) | Detects unused imports, style problems, and common Python issues. It automatically fixes problems when safe to do so. |

### Black

Black is used as the Python formatter.

| Hook | Purpose |
|---|---|
| `black` | Formats Python code consistently and checks that project files follow the expected style. |

### Generic Hygiene Hooks

These hooks are provided by the pre-commit project.

| Hook | Purpose |
|---|---|
| `trailing-whitespace` | Removes trailing spaces at the ends of lines. |
| `end-of-file-fixer` | Ensures files end with exactly one newline. |
| `mixed-line-ending` | Normalizes line endings to LF. |
| `check-yaml` | Validates YAML files. |
| `check-toml` | Validates TOML files such as `pyproject.toml`. |
| `check-json` | Validates JSON files. |
| `check-merge-conflict` | Detects unresolved Git merge-conflict markers. |
| `check-added-large-files` | Prevents unexpectedly large files from being committed. |
| `check-ast` | Verifies that Python files contain valid syntax. |
| `debug-statements` | Detects leftover `breakpoint()` and `pdb` statements. |
| `check-executables-have-shebangs` | Ensures executable files contain a valid shebang. |
| `detect-private-key` | Helps prevent SSH or PEM private keys from being committed. |

## Local Pytest Hook

The existing pre-push hook runs the `session_2` test suite:

```bash
cd session_2 && pytest
```

It runs at the `pre-push` stage rather than on every commit because running tests
is slower than static linting and formatting checks.

The Mini Project root test suite is developed later in Module 1 under:

```text
tests/
```

## Module 1 Package Structure

The baseline notebook is progressively refactored into the following modules:

```text
src/prodml/
├── __init__.py
├── config.py       # paths, hyperparameters, ports, environment configuration
├── data.py         # Parquet loading and train/validation split
├── features.py     # duration cleaning and feature engineering
├── train.py        # model training, evaluation, and persistence
├── predict.py      # model loading and single/batch prediction
└── api/
    └── __init__.py
```

Responsibilities are intentionally separated so that data loading, feature
engineering, training, prediction, configuration, and serving can evolve
independently.

## Configuration

Application configuration is managed through `pydantic-settings`.

Defaults are defined in:

```text
src/prodml/config.py
```

and can be overridden through environment variables using the `PRODML_` prefix.

For example:

```bash
PRODML_API_PORT=9000 python -c "from prodml.config import settings; print(settings.api_port)"
```

or:

```bash
PRODML_RANDOM_STATE=123 python -c "from prodml.config import settings; print(settings.random_state)"
```

This avoids hardcoded machine-specific paths and settings.

## Model Artifacts

The original notebook baseline is stored as:

```text
models/baseline.pkl
```

The refactored training package produces:

```text
models/model.pkl
```

The artifact contains both:

```text
DictVectorizer
LinearRegression
```

so the same fitted feature vocabulary can be reused during prediction.

> Pickle files must only be loaded from trusted sources because Pickle
> deserialization can execute arbitrary Python code.

## Maintenance

Update pre-commit hook versions when needed with:

```bash
pre-commit autoupdate
```

Before committing Mini Project changes, run:

```bash
ruff check src tests
black --check src tests
python -m prodml.train
```

The refactored training workflow should continue to reproduce the notebook
baseline MAE within the required tolerance.
