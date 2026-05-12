# Root — constraints-termux.txt

# constraints-termux.txt

The `constraints-termux.txt` file is a specialized requirements constraint file designed to ensure the stability of the Hermes Agent when deployed within a **Termux** environment on Android. 

Unlike standard requirement files that define what must be installed, this constraints file restricts the versions of dependencies to "known-good" releases that are compatible with the unique limitations of the Android/Termux ecosystem.

## Purpose and Rationale

Developing and running Python applications on Android via Termux presents specific challenges:
1.  **Binary Compatibility:** Many upstream packages release wheels (pre-compiled binaries) for Linux, macOS, and Windows, but rarely for Android/Termux.
2.  **Compilation Hurdles:** Installing from Source Distributions (`sdist`) often fails on-device due to missing build toolchains (Clang, Fortran, etc.) or incompatible C-extensions.
3.  **Upstream Velocity:** Rapid updates to the IPython/Jupyter stack frequently introduce dependencies that require newer system libraries than those available in the current Termux stable repositories.

This file acts as a stabilizer, pinning the interactive execution stack to versions that have been verified to install and run correctly on Android.

## Usage

To install the Hermes Agent with Termux-specific protections, use the `-c` (constraints) flag during installation:

```bash
python -m pip install -e '.[termux]' -c constraints-termux.txt
```

By using this command, `pip` will respect the version limits defined in the file even if the `setup.py` or `pyproject.toml` allows for newer versions.

## Dependency Breakdown

The constraints primarily target the interactive shell and code analysis stack:

| Package | Constraint | Reason |
| :--- | :--- | :--- |
| `ipython` | `<10` | Prevents breaking changes in the REPL environment. |
| `jedi` | `>=0.18.1, <0.20` | Ensures stable autocompletion without requiring incompatible `parso` versions. |
| `parso` | `>=0.8.4, <0.9` | Grammar parser compatible with the pinned `jedi` version. |
| `stack-data` | `>=0.6, <0.7` | Manages traceback formatting; newer versions often require complex dependencies. |
| `pexpect` | `>4.3, <5` | Critical for managing sub-processes and terminal interactions on Android. |
| `matplotlib-inline` | `>=0.1.7, <0.2` | Provides inline plotting support for the Hermes execution environment. |
| `asttokens` | `>=2.1, <3` | Required for AST-based code instrumentation and analysis. |

## Maintenance Note

When upgrading the Hermes Agent's core dependencies, this file should be audited. If a developer attempts to install a new feature that requires a version higher than what is pinned here, they must verify that the new version:
1.  Has a compatible wheel for `aarch64` (if applicable).
2.  Can be compiled from source using the standard `pkg install build-essential` toolchain in Termux.
3.  Does not depend on system-level libraries (like `glibc`) that are absent in the Android Bionic environment.