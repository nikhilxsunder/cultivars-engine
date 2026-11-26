# Changelog

All notable changes to cultivars-engine will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).
The engine is in its `0.0.x` series: `_core` and `_internals` are private and
may change in any release, so every entry that renames, moves or removes a
name is listed under Changed or Removed, whether or not the name is public.
Each cultivars release pins one engine release exactly; the pairing is
recorded at the top of each entry.

## [Unreleased]

### Added

- `README.md`, `CONTRIBUTING.md`, `SECURITY.md`, `ACCESSIBILITY.md`,
  `ROADMAP.md` and `CITATION.cff` for this repository. Until now they were
  empty placeholders.
- Logo and banner artwork for the engine under `assets/`, with a variant
  for dark backgrounds.

### Fixed

- `_VectorResult.nobs` is annotated `int`. It was annotated `float` and
  has always held an `int` (#142).
- `*.pxd` is marked binary in `.gitattributes` and `assets/` is excluded
  from the pre-commit hooks, so the whitespace hooks no longer rewrite the
  Pixelmator sources as text.

## [0.0.3] - 2026-11-23

Pinned by cultivars 1.0.0a6.

### Changed

- `_core` and `_internals` are top-level subpackages. Import
  `cultivars_engine._core` and `cultivars_engine._internals` where 0.0.2
  had `cultivars_engine.engine._core` and
  `cultivars_engine.engine._internals`.
- `cultivars_engine.summary` and `cultivars_engine.typing` import from the
  new locations. Their public names are unchanged.

### Removed

- The `cultivars_engine.engine` subpackage.

### Fixed

- `cultivars_engine.__all__` lists the five modules the distribution
  ships: `_core`, `_internals`, `exceptions`, `summary` and `typing`. In
  0.0.1 and 0.0.2 it also listed `bayes`, `data`, `diagnostics`,
  `experimental`, `forecast`, `multivariate`, `spectral`, `state_space`
  and `univariate`, which belong to cultivars and are not in this package,
  so `from cultivars_engine import *` raised `AttributeError`.

## [0.0.2] - 2026-11-23

Not pinned by any cultivars release.

### Removed

- The `ns-sdn==1.0.0a1` requirement. The engine does not import `ns-sdn`.

## [0.0.1] - 2026-11-23

Not pinned by any cultivars release.

### Added

- First release. The private engine of cultivars, extracted from the
  `cultivars.engine` subpackage as it stood after cultivars 1.0.0a5, as a
  separate distribution with the import name `cultivars_engine`:
  `engine._core` (numerical primitives) and `engine._internals` (model
  machinery).
- `cultivars_engine.exceptions`, `cultivars_engine.summary` and
  `cultivars_engine.typing`, the three public modules that cultivars
  re-exports under `cultivars.exceptions`, `cultivars.summary` and
  `cultivars.typing`.
- `pandas` and `polars` extras for the dataframe conversions.
