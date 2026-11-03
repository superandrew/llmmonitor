+++
id = "002"
title = "Collector uses the Apple GPU"
edit = ["llmmonitor/hardware.py", "tests/test_apple_gpu.py"]
read = ["AGENTS.md", "llmmonitor/apple_gpu.py", "llmmonitor/exporter.py"]
checks = [".venv/bin/python -m unittest tests.test_apple_gpu -v", ".venv/bin/python scripts/check.py", "git --no-pager diff --check HEAD"]
commit = "feat: collector reports the Apple GPU"
edit_format = "diff"
+++

# 002 - Collector uses the Apple GPU

## llmmonitor/hardware.py

Two changes only:

1. Below `from .common import console_options` add `from . import apple_gpu`.
2. In `Sampler.sample`, right before `return {"values": values, "gpus": gpu_rows, ...}`, add:

        if not gpu_rows:
            gpu_rows = apple_gpu.sample()

## tests/test_apple_gpu.py

Add one test class `CollectorTests` with one test. It patches
`llmmonitor.hardware.shutil.which` to return `None` and `llmmonitor.hardware.apple_gpu.sample`
to return `[{"index": "0", "name": "Apple M3 Max", "utilization_ratio": 0.5}]`, then checks that
`Sampler().sample()["gpus"]` equals that same list. Skip the test with
`unittest.skipUnless(platform.system() == "Darwin", ...)`, because `Sampler` needs `psutil` there.
