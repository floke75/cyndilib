# cyndilib – Agent Onboarding Guide

Welcome to **cyndilib**, a Cython-powered wrapper around the NDI® SDK that exposes
high-performance audio/video networking primitives to Python. This repository is
tuned for contributors who need to touch both Python and C/C++-style Cython, so
please read this guide before modifying any files.

---

## 1. Required Background Reading

* **Project overview:** https://cyndilib.readthedocs.io/en/latest/overview.html
* **Development workflow:** https://cyndilib.readthedocs.io/en/latest/development.html
* **API reference (bindings & enums):** https://cyndilib.readthedocs.io/en/latest/reference/index.html
* **Examples and doctests:** https://cyndilib.readthedocs.io/en/latest/examples.html

Consult these pages frequently—this guide summarizes them, but the
documentation remains the source of truth for public behaviour and supported
platforms.

---

## 2. Repository Layout & Key Modules

```
cyndilib/
├── src/cyndilib/           # Core package; mostly .pyx/.pxd Cython bindings
│   ├── *.pyx               # Public API types (Receiver, Sender, Frames, Locks, etc.)
│   ├── *.pxd               # Declarations used across modules
│   └── wrapper/            # Low-level C FFI shims for the NDI® SDK headers & libs
├── tests/                  # Pytest suite; requires compiled extension + test helpers
├── doc/                    # Sphinx documentation mirroring the public API
├── examples/               # Command-line demos (Click-based) showing practical flows
├── build_tests.py          # Helper to build Cython fixtures required by tests
├── cyclean.py              # Utility for removing generated artifacts
└── annotate_index.py       # Annotated HTML index helper for annotated Cython builds
```

**Where to implement logic:**
* Pure Python utilities live in `src/cyndilib/__init__.py` or nearby modules.
* Performance-sensitive code should remain in Cython (`*.pyx` + matching `*.pxd`).
* Raw SDK interoperability belongs under `src/cyndilib/wrapper/` (mirrors the C
  types shown in the "Wrapper" section of the docs).

When touching any module, open its companion page under
[Reference → cyndilib](https://cyndilib.readthedocs.io/en/latest/reference/index.html)
to understand the existing public contract, constructors, and threading rules.

---

## 3. Build & Development Workflow

1. **Install editable build with development extras** (Python ≥ 3.9):
   ```bash
   uv pip install -e .[dev]
   # or: pip install -e .[dev]
   ```
   `uv` is bundled via `uv.lock` but plain `pip` also works.

2. **Recompile Cython extensions** whenever `.pyx`/`.pxd` changes:
   ```bash
   python setup.py build_ext --inplace --parallel "auto"
   ```
   Add `--annotate` to generate HTML performance reports (see
   `doc/source/development.rst`).

3. **Build test fixtures** prior to running the suite:
   ```bash
   python build_tests.py
   ```

4. **Run tests** (uses pytest-xdist + doctests by default):
   ```bash
   py.test
   ```
   *The default options come from `pyproject.toml` (`-n auto`, doctests enabled,
   flaky reporting disabled). Re-run the build step if you see missing symbol
   errors.*

5. **Optional checks:**
   * Documentation: `cd doc && make html`
   * Clean generated artifacts: `python cyclean.py`

Refer to the
[Development documentation](https://cyndilib.readthedocs.io/en/latest/development.html)
for more context on each step, parallel compilation, and profiling flags.

---

## 4. Coding Guidelines

### Python & Cython Style
* Follow **PEP 8** for Python and match the existing indentation/spacing in
  Cython modules (4 spaces, no tabs, descriptive `cdef` names).
* Keep `cdef`/`cpdef` signatures typed. Prefer `nogil` and `noexcept` where the
  current codebase uses them—scan sibling functions for conventions.
* When exposing new public classes or functions, update `__all__` and ensure
  docstrings mirror the style seen in `receiver.pyx` and other modules.
* Reuse enums and structs from `src/cyndilib/wrapper/*.pxd` instead of inventing
  duplicates. If the SDK introduces new symbols, extend the wrapper layer first.
* Threading-sensitive code must coordinate with the locks utilities in
  `locks.pyx`; follow the existing `RLock`/`Condition` patterns.

### Documentation & Examples
* Every public API change should be reflected in `doc/source/reference/` and in
  at least one doctest or example. Most reference pages rely on `autodoc`, so
  make sure docstrings compile without warnings.
* Examples use `click`; keep commands idempotent and guard hardware-dependent
  features behind explicit CLI arguments.

### Testing Expectations
* Pytest is configured to run doctests automatically. Keep snippets under
  `doc/source/**/*.rst` executable and fast.
* If you create new Cython support files for tests, add them to
  `build_tests.py` so `cibuildwheel` can build them in CI.
* Integration tests that require actual NDI® network endpoints should be marked
  with `pytest.importorskip` or guarded by environment variables.

---

## 5. Platform & SDK Notes

* The bindings assume the official NDI® redistributables are available. Binary
  blobs live under `src/cyndilib/wrapper/bin/` for supported platforms.
* Keep cross-platform builds in mind: Windows, macOS (x86_64/arm64 via the
  `constraints-macosx.txt`), and Linux (glibc-based). Avoid POSIX-only APIs in
  shared code.
* Be mindful of the licensing obligations described in
  [`libndi_licenses.txt`](libndi_licenses.txt) and the official SDK docs.

---

## 6. Pull Requests & Review Checklist

Before marking a task complete:
1. Re-run the build/test steps above.
2. Update Sphinx docs and doctests if public behaviour changes.
3. Note any requirement for additional SDK assets or environment variables in
   commit messages and PR descriptions.
4. Keep commits focused; avoid bundling generated C files (they are ignored).

Thank you for contributing to **cyndilib**! This guide should give you the
confidence to explore the bindings and extend them safely while matching the
expectations documented on Read the Docs.
