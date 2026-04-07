# PyCG - Practical Python Call Graphs

[![Linters](https://github.com/vitsalis/PyCG/actions/workflows/linters.yml/badge.svg)](https://github.com/vitsalis/PyCG/actions/workflows/linters.yml)
[![Tests](https://github.com/vitsalis/PyCG/actions/workflows/test.yaml/badge.svg)](https://github.com/vitsalis/PyCG/actions/workflows/test.yaml)

> **This is a community-maintained fork** of [vitsalis/PyCG](https://github.com/vitsalis/PyCG),
> fixing compatibility issues with **Python 3.11 and Windows**.

PyCG generates call graphs for Python code using static analysis.
It efficiently supports:

- Higher order functions
- Twisted class inheritance schemes
- Automatic discovery of imported modules for further analysis
- Nested definitions

You can read the full methodology as well as a complete evaluation on the
[ICSE 2021 paper](https://arxiv.org/pdf/2103.00587.pdf).

You can cite PyCG as follows.
Vitalis Salis, Thodoris Sotiropoulos, Panos Louridas, Diomidis Spinellis and Dimitris Mitropoulos.
PyCG: Practical Call Graph Generation in Python.
In *43rd International Conference on Software Engineering, ICSE '21*,
25–28 May 2021.

---

## Fixes in this fork

| # | Bug | Affected file | Fix |
|---|-----|---------------|-----|
| 1 | `ModuleNotFoundError: No module named 'pkg_resources'` | `pycg/formats/fasten.py` | Replaced `pkg_resources.Requirement` with `packaging.requirements.Requirement` |
| 2 | `ImportManagerError: Can't add edge to a non existing node` | `pycg/machinery/imports.py` | Node is now created **before** the edge in `CustomLoader.__init__` |
| 3 | Same error triggered by `importlib.invalidate_caches()` on Python 3.11+ | `pycg/machinery/imports.py` | Original path hooks are restored before cache invalidation to prevent `CustomLoader` from intercepting system modules |
| 4 | `No module named pycg` on Windows (case-sensitive install) | `pyproject.toml` | Added `[tool.hatch.build.targets.wheel] packages = ["pycg"]` to force lowercase package directory |

---

## Installation

**From this fork (recommended):**

```bash
pip install git+https://github.com/WuWuyn/PyCG.git
```

**From source:**

```bash
git clone https://github.com/WuWuyn/PyCG.git
cd PyCG
pip install -e .
```

> `packaging` is automatically installed as a dependency.

---

## Usage

### Linux / macOS

```bash
# Analyze a package — all .py files discovered automatically
pycg --package mypackage $(find mypackage -type f -name "*.py") -o callgraph.json
```

### Windows (PowerShell)

```powershell
# Run from inside your project directory
cd path\to\your\project

python -m pycg --package . (Get-ChildItem -Recurse -Filter "*.py" | ForEach-Object { $_.Name }) -o callgraph.json
```

---

## Output Format

### Simple JSON (default)

Each key is a node (module or function). Its value is the list of nodes it calls.

```json
{
    "main": ["main.run"],
    "main.run": ["main.greet"],
    "main.greet": ["utils.helper"],
    "utils.helper": ["<builtin>.print"],
    "<builtin>.print": []
}
```

### FASTEN format

```bash
pycg --package mypackage --fasten \
     --product "mypackage" --forge "PyPI" \
     --version "1.0" --timestamp 1234567890 \
     mypackage/module.py -o callgraph.json
```

---

## CLI Reference

```
usage: pycg [-h] [--package PACKAGE] [--fasten] [--product PRODUCT]
            [--forge FORGE] [--version VERSION] [--timestamp TIMESTAMP]
            [--max-iter MAX_ITER] [--operation {call-graph,key-error}]
            [--as-graph-output AS_GRAPH_OUTPUT] [-o OUTPUT]
            [entry_point ...]
```

| Argument | Description |
|----------|-------------|
| `entry_point` | One or more `.py` files to analyze |
| `--package` | Root directory of the package being analyzed |
| `--fasten` | Output in FASTEN format instead of simple JSON |
| `--max-iter` | Max analysis iterations (default: until fixpoint) |
| `--operation` | `call-graph` (default) or `key-error` |
| `-o OUTPUT` | Output file path |

---

## Running Tests

```bash
pip install mock
make test
```

---

## Original Paper

Vitalis Salis, Thodoris Sotiropoulos, Panos Louridas, Diomidis Spinellis and Dimitris Mitropoulos.
PyCG: Practical Call Graph Generation in Python.
In *43rd International Conference on Software Engineering, ICSE '21*, 25–28 May 2021.