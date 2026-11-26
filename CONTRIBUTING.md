# Contributing to cultivars-engine

cultivars-engine is the private numerical layer of [cultivars](https://github.com/nikhilxsunder/cultivars). It has one consumer, and that consumer pins it exactly. Contributing here is therefore different from contributing to a library with a public API: almost every name is private, and a change is correct only if the cultivars release that will pin it still works.

This file covers what is specific to the engine. Conventions shared with cultivars (docstring style, typing, error handling) are restated briefly so this file stands alone.

## Table of Contents

- [Where a Change Belongs](#where-a-change-belongs)
- [Development Setup](#development-setup)
- [Layering Rules](#layering-rules)
- [Coding Standards](#coding-standards)
- [Static Analysis](#static-analysis)
- [Testing](#testing)
  - [Current State](#current-state)
  - [Running a Change Against cultivars](#running-a-change-against-cultivars)
  - [What a New Test Should Look Like](#what-a-new-test-should-look-like)
- [Changes That Cross the Package Boundary](#changes-that-cross-the-package-boundary)
- [Submitting Code Changes](#submitting-code-changes)
- [Pull Request Process](#pull-request-process)
- [Continuous Integration](#continuous-integration)
- [Documentation](#documentation)
- [Release Process](#release-process)
- [License](#license)
- [Security Vulnerability Reporting](#security-vulnerability-reporting)
- [Code of Conduct](#code-of-conduct)
- [Governance Model](#governance-model)
- [Contact](#contact)

## Where a Change Belongs

| The change                                                                                   | Repository                                                                               |
| -------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------- |
| A public model, result, test or forecast function: its arguments, its summary, its docstring | [cultivars](https://github.com/nikhilxsunder/cultivars)                                  |
| A new model family or estimator a user would import                                          | cultivars, with the machinery it needs proposed here in a paired pull request            |
| A numerical primitive, validator, sampler, filter, smoother, objective, solver or record     | cultivars-engine                                                                         |
| `exceptions`, `summary`, `typing`                                                            | cultivars-engine. These are public API of cultivars; see the boundary section below      |
| A user-facing bug whose cause you have not located                                           | An issue on cultivars. It is transferred here if the cause turns out to be in the engine |

Before opening an issue here, search both trackers. Include the engine version (`python -c "import importlib.metadata as m; print(m.version('cultivars-engine'))"`), the cultivars version if it is installed, Python, `numpy` and `scipy` versions, and a minimal snippet. For a numerical discrepancy, include the reference implementation and the values you compared against.

## Development Setup

The project uses [uv](https://docs.astral.sh/uv/).

1. **Prerequisites.** Python 3.12 or later, `uv`, Git. `.python-version` pins the interpreter the maintainer develops on (3.14); the package supports 3.12 upward, so do not use syntax or standard-library features newer than 3.12.

2. **Clone and sync.**

   ```bash
   git clone https://github.com/nikhilxsunder/cultivars-engine.git
   cd cultivars-engine
   git switch dev

   uv sync --all-groups
   uv run pre-commit install
   ```

3. **Optional extras.** The dataframe conversions need `pandas` or `polars`. `uv sync --all-groups` installs both through the `extras` group.

The engine performs no network or file I/O and needs no credentials or configuration.

## Layering Rules

The engine has two private layers and three public modules. The rules are enforced in review.

1. **Imports run downward only.** `_core` imports from `numpy`, `scipy`, the standard library and `exceptions`. `_internals` imports from `_core` and `exceptions`. `_core` never imports from `_internals`, and neither imports from `cultivars`. `pandas` and `polars` are imported lazily, behind `TYPE_CHECKING` and `_core._loaders._require_optional`, and never at module import time.
2. **`_core` knows nothing about a model.** A symbol belongs in `_core` when it has more than one consumer and can be stated without reference to a model class: a validator, a matrix routine, a transform and its inverse, a single conditional draw, a value object. Everything that knows what a model is belongs in `_internals`.
3. **`_internals` is organized by role.** Records (frozen dataclasses that carry state between layers), interfaces that outside code implements, machinery (substrates, filters, smoothers, objectives, solvers, simulators), and bases (the protected classes the public models subclass and the mixins results take behavior from). The package docstring of `_internals` lists which module holds which. Put a new symbol in the module whose role it shares before creating a new module.
4. **Everything is underscore-private.** Modules, functions and classes in both layers carry a leading underscore. cultivars binds the public name when it subclasses or re-exports. The exceptions are the structural protocols in `_core._protocols` and the three interfaces outside code implements (`MeanPredictor`, `MeanFunctionEngine`, `NumpyMLPEngine`).
5. **A result is a record.** Fit, posterior and result objects are frozen dataclasses with every field annotated and documented. They hold what was estimated and what is needed to forecast or simulate from it, and no reference back to the model that produced them.
6. **The three public modules are thin.** `summary` and `typing` bind names defined in `_core`; `exceptions` defines the hierarchy and nothing else. Do not add logic to them.

## Coding Standards

- **Module header.** Every `.py` file begins with the MIT header (copyright 2026 Nikhil Sunder), then the module docstring, then `from __future__ import annotations`.
- **Style.** `ruff format` and `ruff check` with the rule set in `pyproject.toml` (`E`, `F`, `W`, `I`, `B`, `UP`, `SIM`, `RUF`, `ANN`, `D`; line length 100).
- **Types.** Complete annotations on every parameter and return value, private or not. Respect superclass contracts, use proper overloads, and do not weaken `Optional` defaults. The package ships `py.typed`, so cultivars and its users type-check against these annotations.
- **Docstrings.** Google style on every module, class and function, with a doctest example where it clarifies behavior. Document parameters, return values and every exception raised. Mathematical content uses LaTeX in a raw docstring.
- **Errors.** Validate at the boundary and raise `DimensionError`, `SpecificationError` or `NumericalError` from `cultivars_engine.exceptions`. Do not let a raw `numpy.linalg.LinAlgError` escape where the failure can be named; catch it and raise `NumericalError` with a message that states what was expected and what was got. No bare `except:` and no silent failure.
- **No `assert` for validation.** `python -O` strips assertions. A check that guards an input raises one of the package exceptions.
- **Randomness.** An entry point that draws takes `seed: int | np.random.Generator | None` and builds a generator with `np.random.default_rng(seed)`; every primitive beneath it takes that generator as an `rng` argument. NumPy's global random state (`np.random.seed`, `np.random.normal` and the like) is not used anywhere in the engine and must not be introduced.
- **Dependencies.** The engine's code depends on `numpy` and `scipy`. A new runtime dependency needs an issue and the maintainer's agreement before a pull request.

## Static Analysis

Run these before opening a pull request. All three pass on `dev` and are expected to keep passing.

```bash
uv run ruff check src
uv run ruff format --check src
uv run mypy src
```

The pre-commit hooks run the same checks, plus whitespace and TOML/YAML checks and `prettier` for Markdown and YAML. `assets/` is excluded from the hooks and `*.pxd` is marked binary, because the whitespace hooks corrupt Pixelmator sources.

Any suppressed warning (`# noqa`, `# type: ignore`) carries the specific code and a comment that justifies it.

## Testing

### Current State

This is stated plainly because it determines what a reviewer will ask of you.

- `tests/` is empty. `uv run pytest` collects nothing.
- The engine carries 573 doctest prompts across 31 files. In 9 of those files the examples import from `cultivars`, so the full doctest suite cannot run from this repository alone.
- In practice the engine is tested through cultivars: cultivars' docstring examples exercise the engine on every public path.

Building the engine's own test tree is scheduled work; see [ROADMAP.md](ROADMAP.md). Until it exists, the check below is required for any change to behavior.

### Running a Change Against cultivars

From a cultivars checkout that sits beside this one:

```bash
cd ../cultivars
uv sync --all-groups

# Replace the pinned engine with your working tree
uv pip install -e ../cultivars-engine

# --no-sync keeps uv from reinstalling the pinned release
uv run --no-sync pytest --doctest-modules src/cultivars
```

A plain `uv run` re-syncs the environment and puts the pinned engine back, so keep `--no-sync` on every command until you are done, then run `uv sync --all-groups` to restore the environment.

Report the result in the pull request: the cultivars commit you ran against, and the pass and fail counts before and after your change.

### What a New Test Should Look Like

When you add tests, follow the layout the test tree is being built toward:

- `tests/` mirrors `src/cultivars_engine`: `_core/_transforms.py` is tested by `tests/_core/test_transforms.py`.
- Test success and failure paths. Every validator has a test for each exception it raises.
- Property tests with [Hypothesis](https://hypothesis.readthedocs.io/) for the `_core` primitives, where an invariant says more than a fixed case: a transform followed by its inverse is the identity; companion eigenvalues are the reciprocals of the lag-polynomial roots; a reparameterized coefficient vector is always stationary.
- Reference comparisons against pinned fixtures, not against a library imported at test time.
- Doctest examples import from `cultivars_engine`, never from `cultivars`, so they run here.

`pytest`, `pytest-cov` and `hypothesis` are in the `test` dependency group.

## Changes That Cross the Package Boundary

cultivars imports several hundred private names from the engine. Renaming, moving, removing or changing the signature or behavior of any name cultivars uses breaks cultivars the moment it pins the new engine. Such a change needs all of the following:

1. **A search.** Find every use in cultivars:

   ```bash
   grep -rn "the_name" ../cultivars/src
   ```

2. **A paired pull request in cultivars** that adapts to the change and bumps the pin, opened against cultivars' `dev` and linked from the engine pull request. Neither is merged until both are approved.
3. **A CHANGELOG entry** under Changed or Removed, naming the old and the new spelling. Private names are listed too, because the changelog is how the paired cultivars change is reviewed.
4. **For `exceptions`, `summary` and `typing`, more care.** These are public API of cultivars. A change to a class name, a field, a method signature or an alias's permitted values is a change to cultivars' public surface and follows cultivars' stability policy, not the engine's. Additions are fine; removals and renames need a deprecation in cultivars first.

A change that touches no name cultivars uses, and does not alter numerical output, needs none of this.

## Submitting Code Changes

1. **Fork and clone** the repository.
2. **Branch from `dev`** with a prefixed name: `fix/142-vector-result-nobs-int`, `feature/...`, `docs/...`. Including the issue number helps.
3. **Make focused changes** with clear, logical commits.
4. **Sign off your commits (DCO).** Every commit carries a `Signed-off-by` line certifying that you have the right to contribute the code under the project license:

   ```bash
   git commit -s -m "fix: annotate _VectorResult.nobs as int"
   ```

   This indicates agreement to the [Developer Certificate of Origin](DCO.md). Pull requests with commits that are not signed off will not be merged. Cryptographically signed commits (`-S`) are encouraged in addition.

5. **Open the pull request against `dev`.** `main` moves only at a release.

## Pull Request Process

1. **Describe the change.** State what changed, why, which issue it addresses, and whether it crosses the package boundary. Include the result of the cultivars run described under [Testing](#testing).
2. **Link the issue.** The repository's default branch is `main`, and GitHub applies closing keywords only on merge to the default branch. `Closes #142` in a pull request into `dev` links nothing until `dev` is merged into `main`, so also link the issue from the pull request's sidebar under Development.
3. **Review.** The maintainer reviews. Expect first feedback within one to two weeks.
4. **Merge.** Approved changes are merged into `dev` and ship in the next engine release. You are credited in the Git history and, for a fix, in the CHANGELOG.

## Continuous Integration

The workflows under `.github/workflows` were copied from cultivars when the repository was split and are being repointed. Several still reference cultivars' paths and artifact names, and `docs.yml` is empty. Until that work lands, a red or missing check on a pull request is not by itself a statement about your change, and the local commands under [Static Analysis](#static-analysis) and [Testing](#testing) are the gate a reviewer applies.

## Documentation

- The engine reference will be generated from the docstrings, so the docstring is the documentation. Write it for a reader who has the mathematics but not the codebase.
- Refer to engine modules by their real path, `cultivars_engine._core._validators`, in prose and in Sphinx roles. Paths of the form `cultivars.engine._internals...` predate the split; correct them when you touch a docstring that has one.
- Markdown files use ordered heading levels, descriptive link text, and fenced code blocks with a language. `prettier` formats them on commit.
- See [ACCESSIBILITY.md](ACCESSIBILITY.md) for what applies to rendered output and error messages.

## Release Process

Releases are cut by the maintainer.

1. A pull request into `dev` bumps `__version__` in `src/cultivars_engine/__about__.py` and moves the Unreleased entries in `CHANGELOG.md` under the new version and date.
2. `dev` is merged into `main`.
3. The distribution is built with `uv build` and published to PyPI.
4. A pull request in cultivars bumps `cultivars-engine==X.Y.Z` and adapts to anything listed under Changed or Removed.

Releases 0.0.1 through 0.0.3 were uploaded by the maintainer directly and are not tagged in this repository. Publishing from GitHub Actions through PyPI Trusted Publishing, with attestations and a signed tag per release, is scheduled; see [ROADMAP.md](ROADMAP.md) and [SECURITY.md](SECURITY.md).

Versioning: the engine is in `0.0.x`. Until further notice any release may change any private name, and the exact pin in cultivars is what makes that safe.

## License

By contributing to cultivars-engine, you agree that your contributions will be licensed under the project's [MIT License](LICENSE). Every source file carries the MIT header (copyright 2026 Nikhil Sunder).

## Security Vulnerability Reporting

Do not report security vulnerabilities through public issues. Use "Report a vulnerability" under the repository's Security tab, or email nsunder724@gmail.com. See [SECURITY.md](SECURITY.md).

## Code of Conduct

This project adheres to the [Code of Conduct](CODE_OF_CONDUCT.md). By participating, you are expected to uphold it. Report unacceptable behavior to nsunder724@gmail.com.

## Governance Model

cultivars-engine follows the same centralized governance as cultivars. The project owner and lead, Nikhil Sunder, has final authority on direction, contributions and dispute resolution.

## Contact

Open an issue on [cultivars-engine](https://github.com/nikhilxsunder/cultivars-engine/issues) or [cultivars](https://github.com/nikhilxsunder/cultivars/issues), or write to nsunder724@gmail.com.
