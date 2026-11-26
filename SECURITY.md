# Security Policy

cultivars-engine is the numerical layer of [cultivars](https://github.com/nikhilxsunder/cultivars). The two packages share one maintainer, one reporting route and one response process. This file states what is specific to the engine and is accurate for the releases that exist today.

## Table of Contents

- [Current Status](#current-status)
- [Reporting a Vulnerability](#reporting-a-vulnerability)
- [Response Process](#response-process)
- [Disclosure Policy](#disclosure-policy)
- [Supported Versions](#supported-versions)
- [Threat Model](#threat-model)
- [Vulnerability Classes and Mitigations](#vulnerability-classes-and-mitigations)
- [Security Measures](#security-measures)
- [Software Delivery](#software-delivery)
- [Verifying a Release](#verifying-a-release)
- [Security Updates and Announcements](#security-updates-and-announcements)

## Current Status

As of 2026-11-26 no security advisory has been published for cultivars-engine and the maintainer knows of no unpatched vulnerability in it.

## Reporting a Vulnerability

**Do not report security vulnerabilities through public GitHub issues.**

Report through either of:

- **GitHub private vulnerability reporting** (preferred): the "Report a vulnerability" button under the [Security tab](https://github.com/nikhilxsunder/cultivars-engine/security) of this repository. If you are not sure which package is affected, report against either repository; the report is routed to the right one.
- **Email**: nsunder724@gmail.com

Please include:

- The type of vulnerability and its impact, including how it could be exploited.
- The affected version of `cultivars-engine` and, if relevant, of `cultivars`.
- The source file and location (tag, branch or commit, or a direct URL).
- Step-by-step instructions to reproduce, and proof-of-concept code if you have it.
- Any configuration needed to reproduce.

## Response Process

1. **Acknowledgment** within 48 hours.
2. **Verification** of the vulnerability and its impact.
3. **Remediation**: a fix is developed and tested, here and in cultivars if both are affected.
4. **Disclosure**, coordinated with you once a fix is ready.

## Disclosure Policy

- The aim is a fix within 60 days of verification for any vulnerability of medium or higher severity.
- A fix in the engine reaches users through a new engine release and a cultivars release that pins it. Both are published before the advisory.
- You are credited in the advisory and the `CHANGELOG` unless you ask otherwise.

## Supported Versions

Each cultivars release pins one engine release exactly, so supporting an engine release means supporting the cultivars release that pins it.

| Engine release                                     | Supported |
| -------------------------------------------------- | --------- |
| The release pinned by the latest cultivars release | Yes       |
| The latest engine release, if newer than that      | Yes       |
| Every other release                                | No        |

There are no long-term support branches. A security fix is not backported to an older engine release; it ships in a new one, with a new cultivars release to pin it.

## Threat Model

The engine is a numerical compute library, not a service or a client. It receives in-memory arrays, performs linear algebra, optimization and sampling, and returns frozen records.

- **No network, no files, no credentials.** The engine opens no sockets, reads and writes no files, spawns no processes and reads no environment variables or secrets. Its source uses none of `socket`, `urllib`, `subprocess`, `pickle`, `eval` or `exec`.
- **The attack surface is the data and the specification.** The realistic risks are resource exhaustion from an adversarially large specification, and malformed arrays reaching linear algebra.
- **Deserialization is the caller's risk.** The engine's records are ordinary dataclasses and can be pickled by the caller. The engine never unpickles anything itself. Loading a pickled result from an untrusted source executes arbitrary code, as with any pickle, and must not be done.
- **The dependency is imported, not sandboxed.** A compromised release of the engine, or of `numpy` or `scipy`, runs with the privileges of the importing process. Delivery integrity is therefore the main control; see [Software Delivery](#software-delivery).

## Vulnerability Classes and Mitigations

1. **Resource exhaustion.** Extreme lag orders, system dimensions or draw counts can exhaust memory or CPU. Validators in `_core._validators` check orders, shapes and counts for consistency and raise `SpecificationError` or `DimensionError` before work is allocated. They enforce floors and conformability, not a general ceiling on problem size; a caller that accepts specifications from an untrusted party must bound them itself.
2. **Malformed or non-finite input.** Rank, shape and finiteness are checked where data enters, and a failure raises a specific exception. The package does not continue on bad input.
3. **Information in error messages.** The convention at every raise site is to state the argument, what was expected and what was got, as a name, a shape or a scalar. Data arrays are not echoed into messages.
4. **Dependency vulnerabilities.** The engine's code imports only `numpy` and `scipy`, with `pandas` and `polars` optional. Dependabot is configured for the Python dependencies and the GitHub Actions in this repository, and a dependency-review workflow runs on pull requests. The 0.0.3 metadata also declares `fedfred` and `matplotlib`, which the engine does not import. They are installed with it all the same, so they are part of what an install of the engine brings in.
5. **Build and release supply chain.** See the next two sections for what is and is not in place.

## Security Measures

In place today:

- Complete type annotations, checked with `mypy` by the pre-commit hooks.
- `ruff` linting, including the bugbear (`B`) rules.
- CodeQL, dependency-review and OpenSSF Scorecard workflows, and a Dependabot configuration.
- Private vulnerability reporting enabled on the repository.
- Explicit randomness: entry points take a `seed` (an integer, a `numpy.random.Generator` or `None`) and build a generator from it, primitives take that generator as an `rng` argument, and NumPy's global random state is never read or seeded.

Not yet in place:

- The engine has no test suite of its own, so there is no automated regression check in this repository.
- The lint, test and release workflows were copied from cultivars and are being repointed. Until then they do not gate changes here.
- `bandit` and `pip-audit` are in the `lint` dependency group but are not run automatically.

## Software Delivery

| Channel         | State                                                                                                                                                                                                         |
| --------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **PyPI**        | The only distribution channel. All traffic is over TLS. Releases 0.0.1, 0.0.2 and 0.0.3 were built and uploaded by the maintainer and carry **no** [PEP 740](https://peps.python.org/pep-0740/) attestations. |
| **Git tags**    | No engine release is tagged in this repository yet.                                                                                                                                                           |
| **conda-forge** | Not published.                                                                                                                                                                                                |

Planned, and tracked in [ROADMAP.md](ROADMAP.md): publishing exclusively from a GitHub Actions workflow through PyPI Trusted Publishing (OIDC), which removes long-lived upload tokens and attaches build-provenance attestations to every file, and a signed tag for every release. This section will be updated when the first release ships that way.

## Verifying a Release

Until attestations exist, the available check is the file hash.

1. **Pin with hashes.** `uv lock` records the SHA-256 of every file it resolves, and `uv sync --locked` refuses a file that does not match. With pip, use a requirements file with `--require-hashes`.
2. **Compare against PyPI.** The SHA-256 of each file is shown under "Download files" on the [project page](https://pypi.org/project/cultivars-engine/#files). To compute it locally:

   ```sh
   pip download --no-deps "cultivars-engine==0.0.3"
   shasum -a 256 cultivars_engine-0.0.3-py3-none-any.whl
   ```

A hash confirms that the file you have is the file PyPI serves. It does not establish which commit or workflow built it; that is what attestations will add.

## Security Updates and Announcements

- GitHub Security Advisories on this repository and on cultivars.
- The `CHANGELOG` of both packages.
- The release notes of the cultivars release that pins the fixed engine.

---

For questions about this policy, contact nsunder724@gmail.com.
