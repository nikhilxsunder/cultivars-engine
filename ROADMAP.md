# Roadmap

cultivars-engine has no model roadmap of its own. It exists to serve [cultivars](https://github.com/nikhilxsunder/cultivars), and what cultivars intends to ship is stated in [its roadmap](https://github.com/nikhilxsunder/cultivars/blob/main/ROADMAP.md). This file states what the engine has to deliver for each cultivars milestone, and the infrastructure this repository still needs now that it stands alone. Releases are gated on exit criteria, not dates.

Issue-level tracking lives in the [v1.0.0b1 milestone](https://github.com/nikhilxsunder/cultivars-engine/milestone/1), which mirrors the milestone of the same name in cultivars.

Last updated: 2026-11-26

## Status

| Engine release                     | For                                 | State                                                     |
| ---------------------------------- | ----------------------------------- | --------------------------------------------------------- |
| 0.0.1, 0.0.2, 0.0.3                | The split out of `cultivars.engine` | Released 2026-11-23. 0.0.3 is pinned by cultivars 1.0.0a6 |
| Further 0.0.x releases             | Remaining cultivars alphas          | As fixes accumulate                                       |
| The release cultivars 1.0.0b1 pins | Validated core                      | In progress                                               |
| The release cultivars 1.0.0 pins   | Stable                              | Planned                                                   |
| The release cultivars 1.1.0b1 pins | Compute-engine contract             | Planned                                                   |

The engine's version number is independent of cultivars'. This table is keyed to the cultivars release an engine release serves, because that is what determines its content.

## Principles

1. **The engine follows cultivars.** Nothing model-level lands here without a consumer in cultivars, in the same release cycle.
2. **Private stays private.** `_core` and `_internals` carry no compatibility promise. The exact pin in cultivars is the mechanism that makes this safe, and it stays exact.
3. **Three public modules, and only three.** `exceptions`, `summary` and `typing` are public API of cultivars and follow its stability policy.
4. **Two runtime dependencies in engine code.** `numpy` and `scipy`. Dataframe support stays optional.
5. **The engine can be checked on its own.** Linting, typing, tests, doctests and documentation run from this repository without cultivars installed. This is not yet true; making it true is most of the work below.
6. **Criteria, not dates.**

## For cultivars 1.0.0b1: an engine that stands on its own

The split moved the code. The beta is where the repository around it catches up.

**Correctness.** The open issues in the v1.0.0b1 milestone (67 as of this writing). Enhancements are deferred until after cultivars 1.0.0.

**Tests.**

- `tests/` mirrors `src/cultivars_engine`. It is empty today.
- Unit and Hypothesis property tests on the `_core` primitives: companion roots, difference and undifference round trips, reparameterizations, and every validator's failure paths. The cultivars roadmap names these tests and its 100% branch-coverage floor on the primitives; the primitives live here, so the tests do.
- Unit tests on the `_internals` machinery that can be stated without a public model: the Kalman filter and smoothers against closed-form cases, the Hamilton filter and Kim smoother, the samplers against known moments.
- Reference comparisons stay in cultivars, where the public models are.

**Doctests that run here.** 9 of the 31 files with examples import from `cultivars`. Those examples are rewritten against the engine, or moved to the public class in cultivars that they actually demonstrate, so that `pytest --doctest-modules src/cultivars_engine` passes in this repository.

**Reference sweep.** What the split left behind:

- 173 mentions of `cultivars.engine._core` and `cultivars.engine._internals` in 26 files.
- `# filepath: /src/cultivars/...` header comments in all 59 source files.
- The package docstring of `cultivars_engine/__init__.py`, which is still cultivars' own.
- `description` in `pyproject.toml` and the empty `__description__` and `__summary__` in `__about__.py`.

**Continuous integration.** The workflows were copied from cultivars and several do not yet apply to this package: `analyze.yml` lints `src/fedfred/`, `test.yml` and `reports.yml` measure `src/cultivars`, and `prerelease.yml` and `release.yml` look for `cultivars-*` artifacts. Each is repointed or removed, and the set that remains gates pull requests: `ruff`, `mypy`, tests on Python 3.12 to 3.14, and doctests.

**Documentation.** An engine reference built with Sphinx from this repository's docstrings and deployed once per release to `https://nikhilxsunder.github.io/cultivars-engine/<version>/`. The cultivars site already derives its link and its intersphinx inventory URL from the engine version it has installed, so the only requirement on this side is that every released version has a deployment at that path. `docs.yml` is empty today and there is no `docs/` source tree.

**Release pipeline.** Publishing from GitHub Actions through PyPI Trusted Publishing, with attestations, and a signed `vX.Y.Z` tag for every release. Releases 0.0.1 to 0.0.3 were uploaded by hand and are untagged.

**Exit criteria.** cultivars 1.0.0b1 pins an engine release for which all of the following hold:

1. Its CI passes lint, type check, tests and doctests in this repository alone.
2. Coverage is at the floor cultivars' CONTRIBUTING sets: 100% branch coverage on the `_core` primitives.
3. Its reference is deployed at its version path and resolves from the cultivars site.
4. It was published by the release workflow, with attestations, from a signed tag.
5. No docstring names a `cultivars.engine` path.

## For cultivars 1.0.0: stable

- `exceptions`, `summary` and `typing` are frozen with cultivars' stable core. From cultivars 1.0.0 onward, a breaking change to any of them requires a cultivars major version.
- A beta period in which cultivars' pin moves only for fixes.
- A conda-forge feedstock for the engine, submitted before cultivars' own. cultivars' recipe depends on it, so the engine has to be on conda-forge first.
- Classifier moved off `Development Status :: 3 - Alpha` together with cultivars.
- A decision on the engine's version scheme after 1.0: stay in `0.x` behind the exact pin, or adopt a scheme that signals which cultivars line a release serves. Either way the pin stays exact.

## For cultivars 1.1.0b1: compute-engine contract

cultivars' roadmap schedules pluggable compute engines for 1.1: an `Engine` protocol, capability flags, an entry-point registry with an integer contract version, and the existing NumPy implementation moved behind that contract with numerically identical results. The contract and the NumPy reference implementation are engine code, so that work lands in this repository, followed by the conformance suite that accelerated engines are tested against.

The accelerated engines themselves (MLX, CuPy) ship as separate plugin distributions and are not part of this package.

## Under consideration

Not scheduled. Each needs a decision before it gets a milestone.

- **`ns-sdn` as a second consumer.** `ns-sdn` duplicates several engine primitives and reaches users through `cultivars.experimental`, so reusing the engine's would remove the duplication. It would also mean a second package depends on private names, which the exact pin was designed for one consumer to do. The open question is which names `ns-sdn` may rely on and how its pin tracks cultivars'.
- **Dependency declarations.** The engine declares `fedfred` and `matplotlib` and imports neither. `cultivars.data` imports `fedfred` and cultivars does not declare it; nothing in either package imports `matplotlib` at runtime. Declaring `fedfred` in cultivars and dropping `matplotlib` from the runtime requirements would make an engine-only install smaller and each package's metadata true to its imports. It changes cultivars' metadata, so it is a paired change.
- **A documented extension surface.** The protected base classes in `_internals._models` and the interfaces in `_internals._priors`, `_predictors`, `_engines` and `_specifications` are what a third-party model would build on. Promoting any of them to a supported surface would be considered only for a concrete outside consumer.

## Not planned

- A public modelling API in this package. Models, results and tests are cultivars'.
- A version range in cultivars' pin.
- Deep-learning frameworks as dependencies, as in cultivars' roadmap.

## How this roadmap changes

The file is revised at each engine release that a cultivars release pins, and whenever the milestone's scope changes. Proposals go through an issue; see [CONTRIBUTING.md](CONTRIBUTING.md) for what a change that crosses the package boundary has to include.
