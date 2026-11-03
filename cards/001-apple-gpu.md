+++
id = "001"
title = "Apple GPU sample"
edit = ["llmmonitor/apple_gpu.py", "tests/test_apple_gpu.py"]
read = ["AGENTS.md", "docs/metrics.md"]
checks = ["test -s llmmonitor/apple_gpu.py && test -s tests/test_apple_gpu.py", ".venv/bin/python -m unittest tests.test_apple_gpu -v", ".venv/bin/python scripts/check.py", "git add -N -- llmmonitor/apple_gpu.py tests/test_apple_gpu.py && git --no-pager diff --check HEAD"]
commit = "feat: sample the Apple GPU from ioreg"
edit_format = "whole"
+++

# 001 - Apple GPU sample

On a Mac there is no `nvidia-smi`, so the GPU panels stay empty. `ioreg` reports the Apple GPU
without sudo. This card only reads it; card 002 plugs it into the collector.

## llmmonitor/apple_gpu.py

    def parse_ioreg(text)
    def sample(run=subprocess.run, system=platform.system)

- `parse_ioreg(text)` returns a dict or `None`:
  - `"Device Utilization %"=N` (N integer) -> key `utilization_ratio` with value `N / 100`.
    When this is missing, return `None`.
  - `"In use system memory"=N` -> key `memory_used_bytes` with value `float(N)`. The pattern
    must not match `"In use system memory (driver)"=0`. When missing, leave the key out.
  - `"model" = "Apple M3 Max"` -> key `name` with that string; `"Apple GPU"` when missing.
  - key `index` is always the string `"0"`.
- `sample(run, system)` returns a list:
  - `system()` is not `"Darwin"` -> `[]`, without calling `run`.
  - `run(["ioreg", "-r", "-d1", "-c", "IOAccelerator"], capture_output=True, text=True, timeout=2, check=True)`.
    `OSError` or `subprocess.SubprocessError` -> `[]`.
  - `parse_ioreg(result.stdout)` is `None` -> `[]`, otherwise a list with that one dict.

## tests/test_apple_gpu.py

Use `unittest` and `types.SimpleNamespace`. This module-level constant is the input:

    IOREG = '    "PerformanceStatistics" = {"In use system memory (driver)"=0,"Alloc system memory"=29249503232,"Tiler Utilization %"=63,"Renderer Utilization %"=61,"Device Utilization %"=99,"In use system memory"=25320931328}\n    "model" = "Apple M3 Max"\n'

- `parse_ioreg(IOREG)` equals `{"index": "0", "name": "Apple M3 Max", "utilization_ratio": 0.99, "memory_used_bytes": 25320931328.0}`.
- `parse_ioreg('"model" = "Apple M3 Max"')` is `None`.
- `parse_ioreg('"Device Utilization %"=5')` equals `{"index": "0", "name": "Apple GPU", "utilization_ratio": 0.05}`.
- `sample` with `system=lambda: "Linux"` and a `run` that raises `AssertionError` returns `[]`.
- `sample` with `system=lambda: "Darwin"` and a `run` returning `SimpleNamespace(stdout=IOREG)` returns a list of 1 dict whose `name` is `"Apple M3 Max"`.
- `sample` with `system=lambda: "Darwin"` and a `run` that raises `subprocess.TimeoutExpired("ioreg", 2)` returns `[]`.
- `sample` with `system=lambda: "Darwin"` and a `run` that raises `FileNotFoundError` returns `[]`.
- `sample` with `system=lambda: "Darwin"` and a `run` returning `SimpleNamespace(stdout="")` returns `[]`.
