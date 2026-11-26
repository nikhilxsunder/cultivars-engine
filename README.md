<div align="center">
    <img src="https://raw.githubusercontent.com/nikhilxsunder/cultivars-engine/dev/assets/exported/cultivars_engine_banner.png"  alt="cultivars-engine banner">
</div>

## The numerical engine behind cultivars: primitives, state-space substrates, estimators, samplers and result records, published as its own distribution and pinned exactly by every cultivars release.

|                    |                                                                                                                                                                                                                                                                                                                             |
| ------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Distribution**   | [![PyPI](https://img.shields.io/pypi/v/cultivars-engine.svg)](https://pypi.org/project/cultivars-engine/) [![Python Versions](https://img.shields.io/pypi/pyversions/cultivars-engine.svg)](https://pypi.org/project/cultivars-engine/)                                                                                     |
| **Quality**        | [![Ruff](https://img.shields.io/endpoint?url=https://raw.githubusercontent.com/astral-sh/ruff/main/assets/badge/v2.json)](https://github.com/astral-sh/ruff) [![Checked with mypy](https://www.mypy-lang.org/static/mypy_badge.svg)](https://mypy-lang.org/)                                                                |
| **Packaging**      | [![uv](https://img.shields.io/endpoint?url=https://raw.githubusercontent.com/astral-sh/uv/main/assets/badge/v0.json)](https://github.com/astral-sh/uv) [![Hatch project](https://img.shields.io/endpoint?url=https://raw.githubusercontent.com/pypa/hatch/master/docs/assets/badge/v0.json)](https://github.com/pypa/hatch) |
| **License**        | [![License](https://img.shields.io/pypi/l/cultivars-engine.svg)](https://github.com/nikhilxsunder/cultivars-engine/blob/main/LICENSE)                                                                                                                                                                                       |
| **Usage**          | [![PyPI Downloads](https://static.pepy.tech/badge/cultivars-engine)](https://pepy.tech/projects/cultivars-engine)                                                                                                                                                                                                           |
| **Parent project** | [![cultivars on PyPI](https://img.shields.io/pypi/v/cultivars.svg?include_prereleases&label=cultivars)](https://pypi.org/project/cultivars/) [![DOI](https://zenodo.org/badge/DOI/10.5281/zenodo.23028870.svg)](https://doi.org/10.5281/zenodo.23028870)                                                                    |

> **This is not the package to install for modelling.** Install [`cultivars`](https://github.com/nikhilxsunder/cultivars). It depends on the one engine release it was built against and brings it along. Install `cultivars-engine` directly only to work on the engine, or to build on its internals with an exact pin of your own.

## What it is

cultivars ships as two distributions. `cultivars` holds the public models, results and tests. `cultivars-engine` (import name `cultivars_engine`) holds what sits beneath them: about 42,000 lines of numerical primitives and model machinery that the public classes subclass, call and re-export, plus three small public modules that both layers share.

| Module                        | Status  | Contents                                                                                                                                                                                                                                                                                                                                               |
| ----------------------------- | ------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `cultivars_engine._core`      | Private | 21 modules of primitives that know nothing about any model: input validators, matrix and lag-polynomial routines, estimators, single-draw sampling primitives, stationarity transforms and reparameterizations, spectra, wavelets, scoring rules, the value objects behind every summary, structural protocols, and package-wide defaults              |
| `cultivars_engine._internals` | Private | 31 modules of model machinery: the linear-Gaussian and nonlinear state-space substrates with their filters and smoothers, the Hamilton filter and Kim smoother, optimizer objectives, solvers, priors, simulators, fit, posterior and result records, the protected base classes the public models subclass, and the mixins results take behavior from |
| `cultivars_engine.exceptions` | Public  | `CultivarsError` and its three subclasses `DimensionError`, `SpecificationError` and `NumericalError`, with `StabilityWarning` and `ExperimentalWarning`                                                                                                                                                                                               |
| `cultivars_engine.summary`    | Public  | `SummaryTable`, the record every `summary()` returns, and `InformationCriteria`                                                                                                                                                                                                                                                                        |
| `cultivars_engine.typing`     | Public  | The 19 aliases for categorical arguments (`Trend`, `Method`, `Identification`, `Penalty` and the rest)                                                                                                                                                                                                                                                 |

The dependency runs one way. Engine code imports nothing from `cultivars`.

## How it relates to cultivars

1. **One engine release per cultivars release.** `cultivars` declares `cultivars-engine==X.Y.Z`, never a range.
2. **The pin is exact because the boundary is private.** Of the 380 names `cultivars` 1.0.0a6 imports from the engine, 360 are underscore-private. Private names carry no compatibility promise, so any engine release can break any cultivars release other than the one that pins it.
3. **The two version numbers are independent.** An engine release does not imply a cultivars release, and the engine's version says nothing about cultivars' API.

| cultivars           | cultivars-engine                                       |
| ------------------- | ------------------------------------------------------ |
| 1.0.0a6             | 0.0.3                                                  |
| 1.0.0a5 and earlier | None. The engine was the `cultivars.engine` subpackage |

Engine releases 0.0.1 and 0.0.2 were never pinned by a cultivars release.

## Stability

| Surface                           | Guarantee                                                                                                                                                                                                                                   |
| --------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `exceptions`, `summary`, `typing` | Public. `cultivars.exceptions`, `cultivars.summary` and `cultivars.typing` re-export these modules, and the classes are the same objects under either import path. Their stability follows cultivars' policy. Import them from `cultivars`. |
| `_core`, `_internals`             | None. Names, signatures and behavior can change in any release, including a patch release. Code that imports from them must pin the engine exactly, as `cultivars` does.                                                                    |

Because the classes are defined here, a traceback or a `repr` names `cultivars_engine.exceptions.SpecificationError` even when the import said `cultivars.exceptions`. An `except` clause written against either path catches the same exception.

## Installation

Requires Python 3.12 or later.

For modelling, install cultivars and let it choose the engine:

```bash
pip install --pre cultivars
```

To install the engine on its own, pin it:

```bash
pip install "cultivars-engine==0.0.3"
pip install "cultivars-engine[pandas]==0.0.3"      # or [polars]
```

With `uv`:

```bash
uv add "cultivars-engine==0.0.3"
```

Do not install a different engine version beside an installed `cultivars`. The resolver will refuse, and forcing it leaves a `cultivars` that imports names the engine no longer has.

The engine's own code imports `numpy` and `scipy`, and `pandas` or `polars` only for the `to_pandas()` and `to_polars()` conveniences. The 0.0.3 metadata also declares `fedfred` and `matplotlib`. The engine imports neither. `fedfred` is used by `cultivars.data` and reaches cultivars through this package; `matplotlib` is not imported by either package at runtime.

## A quick check

The public modules work without `cultivars` installed:

```python
>>> from cultivars_engine.exceptions import CultivarsError, SpecificationError
>>> issubclass(SpecificationError, CultivarsError), issubclass(SpecificationError, ValueError)
(True, True)

>>> from cultivars_engine.summary import InformationCriteria, SummaryTable
>>> ic = InformationCriteria.from_likelihood(llf=-120.5, nobs=200, n_params=4)
>>> round(ic.aic, 2), round(ic.bic, 2), round(ic.hqic, 2)
(249.0, 262.19, 254.34)

>>> table = SummaryTable(
...     title="AR(1)",
...     metadata=(("nobs", "200"), ("method", "conditional least squares")),
...     columns=("", "coef", "std err"),
...     rows=(("phi", "0.512", "0.061"),),
...     notes=("Standard errors assume homoskedastic errors.",),
... )
>>> print(table.to_text())
                                     AR(1)
================================================================================
nobs:                              200  method:      conditional least squares
--------------------------------------------------------------------------------
         coef   std err
phi     0.512     0.061
================================================================================
Standard errors assume homoskedastic errors.
```

## Documentation

The engine reference is not published yet. It will be built from this repository and deployed once per release to `https://nikhilxsunder.github.io/cultivars-engine/<version>/`, and the cultivars site links to the reference for the engine release it pins.

Until then:

- Every module, class and function carries a Google-style docstring, readable with `help()` or an editor hover.
- The models built on the engine are documented at <https://nikhilxsunder.github.io/cultivars/stable/> (stable) and <https://nikhilxsunder.github.io/cultivars/dev/> (development).

## Where to report an issue

| What you saw                                                                                          | Where it goes                                                                                                     |
| ----------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------- |
| A wrong number, an error or confusing behavior from a model, result or test in `cultivars`            | [cultivars issues](https://github.com/nikhilxsunder/cultivars/issues/new/choose)                                  |
| A defect you have traced to a file under `src/cultivars_engine`, or a typing or docstring error there | [cultivars-engine issues](https://github.com/nikhilxsunder/cultivars-engine/issues/new/choose)                    |
| A security vulnerability in either package                                                            | Privately, as described in [SECURITY.md](https://github.com/nikhilxsunder/cultivars-engine/blob/main/SECURITY.md) |

If you are not sure, file it against cultivars. Issues are moved between the two trackers as needed, and the labels and milestones match.

## Development

```bash
git clone https://github.com/nikhilxsunder/cultivars-engine.git
cd cultivars-engine
uv sync --all-groups
uv run pre-commit install
uv run ruff check src && uv run ruff format --check src
uv run mypy src
```

The engine has no test tree of its own yet. It is exercised today through cultivars' docstring examples, so a change here should be run against a cultivars checkout before it is proposed. [CONTRIBUTING.md](https://github.com/nikhilxsunder/cultivars-engine/blob/main/CONTRIBUTING.md) gives the commands, the layering rules for `_core` and `_internals`, and what a change that crosses the package boundary has to include. [ROADMAP.md](https://github.com/nikhilxsunder/cultivars-engine/blob/main/ROADMAP.md) lists the test, documentation and CI work scheduled for this repository.

## Citation

The engine is part of cultivars and is not cited separately. If cultivars contributes to published work, cite the concept DOI, which resolves to the latest release:

```bibtex
@software{sunder_cultivars,
  author  = {Sunder, Nikhil},
  title   = {cultivars: Research-grade time series econometrics for Python},
  year    = {2026},
  doi     = {10.5281/zenodo.23028870},
  url     = {https://github.com/nikhilxsunder/cultivars}
}
```

## License

MIT. See [LICENSE](https://github.com/nikhilxsunder/cultivars-engine/blob/main/LICENSE).
