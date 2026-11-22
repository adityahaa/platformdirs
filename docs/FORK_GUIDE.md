# platformdirs fork guide

This checkout belongs to Aditya Hambire’s fork of [tox-dev/platformdirs](https://github.com/tox-dev/platformdirs). The package, architecture, examples and upstream test suite described here are the work of the upstream project and its contributors. This review establishes no fork-specific code contribution.

## Purpose and API design

Desktop and command-line software needs locations for persistent data, settings, disposable cache and runtime files. Those locations follow different operating-system conventions. Platformdirs provides a common Python API that resolves each directory category through the active platform's implementation.

```mermaid
flowchart LR
  A[Application name and options] --> B[PlatformDirs selection]
  B --> C[Windows / macOS / Unix / mobile implementation]
  C --> D[Platform conventions and environment]
  D --> E[Directory string or pathlib.Path]
```

`src/platformdirs/__init__.py` selects the platform class and exposes convenience functions. `api.py::PlatformDirsABC` defines shared options and interfaces. Platform modules supply the actual paths. Android detection considers environment and runtime information, illustrating why selection involves more than a single desktop OS check.

## Example and interpretation

```python
from platformdirs import PlatformDirs

dirs = PlatformDirs("SampleApp", "SampleOrg", version="1")
data = dirs.user_data_path
cache = dirs.user_cache_path
assert data.is_absolute()
assert cache.is_absolute()
```

The result depends on the host's conventions and environment. Data and cache represent different retention policies even when both are writable paths. String properties and `pathlib.Path` properties provide corresponding interfaces. Options such as app author, version, roaming and directory creation affect the generated path or its side effects. Review the platform-specific API before selecting settings for an application.

By default, path lookup avoids creating application directories. `ensure_exists=True` opts into filesystem creation and should be chosen intentionally. The shared implementation also supplies iterators for categories that can have multiple candidate locations. Those distinctions matter when locating existing configuration versus choosing where to write a new file.

## Setup and verification

The checkout declares Python >=3.11. Published usage starts with `python -m pip install platformdirs`. Local development uses `python -m pip install -e .` and the tooling described in `CONTRIBUTING.md` and `tox.toml`. The build uses hatchling and VCS-derived version metadata, so installing from source includes a build step beyond importing the platform modules.

The upstream test tree covers platform-specific behavior and fixtures. A Windows run alone cannot establish Unix or mobile compatibility. This documentation review inspected the selection and shared path code, without executing the cross-platform test matrix. Existing upstream CI badges describe upstream jobs, not a validation result for this fork.

## Source guide and extension points

| Location | Purpose |
|---|---|
| `src/platformdirs/__init__.py` | Class selection and convenience functions |
| `src/platformdirs/api.py` | Shared interface, options and path helpers |
| `src/platformdirs/windows.py` | Windows directory conventions |
| `src/platformdirs/macos.py`, `unix.py` | macOS and Unix conventions |
| `tests/` | Platform and environment behavior fixtures |
| `docs/`, `tox.toml` | Documentation and verification environments |

The library's practical output is an application-specific path derived from platform conventions. A contribution should document its behavior on the affected platform, preserve existing caller semantics, and add focused evidence for the changed convention. This fork's presence alone establishes no individual authorship of that implementation.
