# Rules for the local coding agent (Aider)

llmMonitor monitors local AI: a Python collector and exporter feed Prometheus, and
Grafana draws the dashboards. Read README.md and docs/metrics.md for the context.

1. Work on exactly one card from `cards/`. Read it fully before editing.
   Edit only the files the card lists under `edit`.
2. Python >= 3.10. Only the standard library, plus `psutil` where it is already used.
3. Code that touches the outside world (processes, HTTP, clock) takes the function
   as a parameter with the real one as default, so tests pass fakes.
   New tests never start processes, never open sockets, never sleep.
4. Never invent behaviour the card does not ask for. Unsupported measurements are
   omitted, never replaced with zeros.
5. Small diffs: no reformatting, no renaming, no unrelated clean-up.
   No line may end with spaces.
6. Done means every command in the card's `checks` passes. Never loop on the same
   failure: after two attempts, stop.
7. Do not commit. Andrew commits after review with the card's `commit` message.
