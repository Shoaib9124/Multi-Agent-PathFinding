# PathWeave Development

PathWeave is the project and product name. The Python package is imported as
`pathweave`.

## Requirements

- Python 3.10 or newer.
- NumPy for the MAPPO implementation.
- pytest for the test suite.

## Setup

```bash
python -m venv .venv
# Windows
.\.venv\Scripts\activate
# Linux / macOS
source .venv/bin/activate

python -m pip install --upgrade pip
python -m pip install -r requirements.txt
```

## Run the application

```bash
python main.py
```

Open <http://localhost:3000>. Use `--port 8000` if port 3000 is busy and
`--host 127.0.0.1` to bind to the local machine only.

## Run the tests

```bash
python -m pytest -q
```

## Rebuild the browser bundle

The playground runs the same Python modules through Pyodide. After changing
anything under `pathweave/core`, `pathweave/algorithms`, `pathweave/scenarios.py` or
`pathweave/benchmark.py`, regenerate the bundle:

```bash
python scripts/build_web_bundle.py
```

## Lint

The CI configuration checks for syntax errors and undefined names:

```bash
python -m pip install flake8
flake8 . --count --select=E9,F63,F7,F82 --show-source --statistics
```

## Layout

- `pathweave/core/` — grid, heuristics, problem/solution model, validation, trace.
- `pathweave/algorithms/` — single-agent search, Space-Time A*, prioritized
  planning and CBS.
- `pathweave/rl/` — MAPF environment, observation, reward, networks and MAPPO.
- `pathweave/research.py` — comparison harness, MAPPO runner and batch experiments.
- `main.py` — HTTP server and CLI entry point.
- `.docs/` — static browser application.
- `scripts/` — bundle builder and RL training/evaluation CLI.
- `tests/` — unit and integration tests.
