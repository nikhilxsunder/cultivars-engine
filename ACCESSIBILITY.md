# Accessibility

cultivars-engine is the numerical layer of [cultivars](https://github.com/nikhilxsunder/cultivars). It has no user interface of its own, but it owns three things that people read: the report every fitted result renders itself through, the text of every error and warning the package raises, and the docstrings the reference is generated from. This file states what each of those does today, what has not been checked, and how to report a barrier.

This is a statement of current behavior, not a conformance claim. The engine has not been through a formal audit against WCAG or any other standard, and no automated accessibility testing runs in CI.

The documentation site, the example gallery and guidance on plotting results belong to cultivars and are covered in [its accessibility statement](https://github.com/nikhilxsunder/cultivars/blob/main/ACCESSIBILITY.md).

## Table of Contents

- [Scope](#scope)
- [Reports](#reports)
- [Errors and Warnings](#errors-and-warnings)
- [Docstrings and Reference](#docstrings-and-reference)
- [Known Gaps](#known-gaps)
- [Reporting a Barrier](#reporting-a-barrier)
- [Contributing Accessibly](#contributing-accessibly)

## Scope

| Surface                 | Where it lives                                                       | Audited                                                     |
| ----------------------- | -------------------------------------------------------------------- | ----------------------------------------------------------- |
| Result and test reports | `SummaryTable`, defined in `_core/_containers.py`                    | Reviewed by source reading; no assistive-technology testing |
| Exceptions and warnings | `cultivars_engine.exceptions` and the raise sites across both layers | Reviewed by source reading                                  |
| Docstrings              | Every module, class and function under `src/cultivars_engine`        | Not audited                                                 |
| Engine reference site   | Not published yet                                                    | Not applicable                                              |

The engine draws no figures. No engine module imports a plotting library.

## Reports

Every `summary()` in cultivars returns a `SummaryTable`. The record holds the structure of the report (a title, metadata pairs, one coefficient table, closing notes) separately from its rendering, so one record renders four ways and you can pick the form that works with your tools.

| Renderer        | Output                   | Notes                                                                                                                                   |
| --------------- | ------------------------ | --------------------------------------------------------------------------------------------------------------------------------------- |
| `to_text()`     | Fixed-width plain text   | Rules are drawn with `=` and `-` only. No color, no ANSI escape codes, no box-drawing characters.                                       |
| `_repr_html_()` | HTML `<div>` for Jupyter | Real `<table>`, `<thead>`, `<th>` and `<tbody>` elements. No stylesheet and no JavaScript; all styling is inline. Text is HTML-escaped. |
| `to_pandas()`   | `DataFrame`              | Needs the `pandas` extra.                                                                                                               |
| `to_polars()`   | `DataFrame`              | Needs the `polars` extra.                                                                                                               |

The fields themselves (`title`, `metadata`, `columns`, `rows`, `notes`) are plain tuples of strings and can be read directly without any renderer.

Meaning is never carried by color alone. Significance and model assumptions appear as numbers and as sentences in the notes.

### If you use a screen reader or braille display

Fixed-width text tables rely on column alignment, which linear readers do not convey well. The structured routes are usually easier:

1. Call `to_pandas()` and read the frame row by row, or export it to CSV.
2. In a notebook, use the HTML repr, which exposes table headers to assistive technology.
3. Read `SummaryTable.notes`, which holds the assumptions as plain sentences, and `SummaryTable.rows`, which holds each coefficient row as a tuple.

## Errors and Warnings

The engine defines the hierarchy and composes almost every message cultivars shows.

- **The class says what kind of fix is needed.** `DimensionError` (reshape the data), `SpecificationError` (change the choice), `NumericalError` (look at the numbers). You learn this from the exception's name before reading the message.
- **Messages follow one shape:** what was expected, then what was got, naming the argument.
- **Messages are plain text.** No color, no markup, no terminal escape codes.
- **Warnings are distinct types.** `StabilityWarning` and `ExperimentalWarning` are `UserWarning` subclasses, so they can be filtered, logged or turned into errors by category with the standard `warnings` module.

Known exceptions to this are listed under [Known Gaps](#known-gaps).

## Docstrings and Reference

- Every module, class and function has a Google-style docstring, so the documentation is available as plain text inside Python (`help(obj)`, an editor hover or a language server) without a website.
- Mathematics is written as LaTeX in the docstring source. It is readable as text in a terminal, and will render through MathJax once the reference site exists.
- The reference site is not published yet. When it is, this file will be updated with what was checked on it.

## Known Gaps

These are accurate as of 2026-11-26.

| #   | Gap                                                                                                                                                                            | Effect                                                                                                                                                 |
| --- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------ |
| 1   | The HTML repr hard-codes the color `#555` for metadata labels and notes.                                                                                                       | 7.46:1 on a white notebook, which passes. 2.53:1 on a `#111111` dark background and 2.24:1 on `#1e1e1e`, both below the 4.5:1 threshold for body text. |
| 2   | The HTML repr tables have no `<caption>` and no `scope` attribute on `<th>`, and the first column of each coefficient row is a `<td>`, not a row header.                       | Table purpose and header association depend on the screen reader's inference.                                                                          |
| 3   | The HTML repr sets a fixed monospace font at `0.85em`.                                                                                                                         | Text is smaller than the surrounding notebook text. It scales with browser zoom.                                                                       |
| 4   | Some raw `numpy.linalg.LinAlgError` exceptions escape without being re-raised as `NumericalError`.                                                                             | The message is NumPy's, does not name the argument, and is not caught by `except CultivarsError`.                                                      |
| 5   | A few docstrings are cut off mid-sentence (for example the `Returns` section of `filter` in `_internals/_substrates.py`, which ends at "A").                                   | Incomplete help text, easy to miss with a screen reader.                                                                                               |
| 6   | Many docstrings still name modules by their pre-split path, `cultivars.engine._core` and `cultivars.engine._internals` (173 occurrences in 26 files).                          | Wrong navigation targets in documentation text and, later, broken cross-references on the reference site.                                              |
| 7   | No assistive-technology testing has been done on any surface.                                                                                                                  | Everything above comes from reading source and computing contrast, not from using a screen reader, magnifier or voice control.                         |
| 8   | The import error raised when `pandas` or `polars` is missing is a plain `ImportError`, and its install hint names `cultivars[pandas]` even when the engine is installed alone. | Correct for cultivars users; slightly misleading for anyone using the engine directly.                                                                 |

## Reporting a Barrier

If something the engine produces is hard or impossible to use with your tools, please report it. Include:

1. What you were trying to do and which output, error or docstring was involved.
2. The tool or setup you use (screen reader and browser, notebook front end, terminal, magnifier, and so on).
3. What happened and what you expected.

Ways to reach the maintainer:

- Open an issue on [cultivars](https://github.com/nikhilxsunder/cultivars/issues/new/choose) or [cultivars-engine](https://github.com/nikhilxsunder/cultivars-engine/issues/new/choose) and add "accessibility" to the title. Either tracker is fine; the issue is labelled `accessibility` and moved to wherever the fix belongs.
- Email nsunder724@gmail.com if you would rather not file publicly.

A report is acknowledged, with a statement of whether it could be reproduced and what is planned. This is a small, maintainer-led open-source project, so no timeline is promised, but accessibility reports are treated as bugs, not feature requests.

## Contributing Accessibly

For changes that touch output, messages or docstrings:

- **Report fields and renderers.** Keep meaning in text, not in color or position. Anything added to `to_text()` is added to the HTML repr and the dataframe routes as well, and the reverse.
- **HTML output.** Use semantic elements (`<table>`, `<th>`, `<caption>`), do not hard-code a color that assumes a light background, and emit no JavaScript. Escape every string.
- **Error messages.** Write a complete sentence that names the argument, the expected form and the value received, and raise a package exception class, not a raw library one.
- **Warnings.** Use `StabilityWarning` or `ExperimentalWarning`, or add a new `UserWarning` subclass to `exceptions`, so the category can be filtered. Do not `print`.
- **Docstrings.** Finish every section you start, name modules by their real path, and describe what an example's output shows instead of leaving it to the printed array.

Pull requests that close any item under [Known Gaps](#known-gaps) are welcome.
