# QoLA Agent Guide

QoLA (Quality of Life AITER) is a Python-driven, manifest-based AOT builder for [AITER](https://github.com/ROCm/aiter) MHA kernels. It reads a TOML manifest, clones/patches AITER, and produces either pybind11 Python modules or torch-free C-linkable shared libraries.

## Repository Layout

- `qola/` — main package.
  - `cli.py` — `qola build` entry point.
  - `build_tools/` — orchestration, TOML parsing, AITER namespace resolver, variant matrix expansion.
  - `cpp_itfs/` — C/HIP namespace wrappers and linker version script for symbol isolation.
- `example/` — sample manifests and invocation.
- `patches/aiter/` — AITER patches applied before build.
- `pyproject.toml` — Python packaging and entry points.
- `CLAUDE.md` — existing design notes (use as architecture reference).

## Setup Commands

```bash
# Recommended: use a virtual environment
python -m venv .venv
source .venv/bin/activate

# Install in editable mode
pip install -e .
```

## Build Commands

```bash
# Run with an example manifest
qola build example/qola.toml
```

A manifest declares the AITER commit, target architectures (`gfx90a`, `gfx942`, etc.), MHA variants, and whether to emit `cpp_itfs` (torch-free) or pybind11 modules.

## Test Commands

QoLA currently relies on example builds as integration tests:

```bash
pip install -e .
qola build example/qola.toml
ls example/build*/lib*.so
```

Add unit tests under a `tests/` directory as the project grows; run with `pytest`.

## Lint / Code Style

```bash
ruff check qola/ example/
black --check qola/ example/
```

## Key Conventions

- All AITER symbols are wrapped in a per-library namespace (`QOLA_NS_*` macros) to avoid collisions.
- Patches in `patches/aiter/` are applied deterministically; keep them minimal and upstreamable where possible.
- `cpp_itfs` mode intentionally avoids PyTorch headers for C-linkable `.so` output.

## Common Gotchas

- AITER builds require ROCm/HIP toolchain and matching GPU architecture.
- Building AITER kernels can take tens of minutes; set manifest variants conservatively during iteration.
- The project only supports Python >= 3.10.
