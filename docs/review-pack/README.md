# Pico Review Pack

## Project Summary

Pico is a local terminal coding agent for repository-grounded engineering tasks. The project combines workspace-aware prompting, constrained tools, structured memory, checkpoint / resume, and auditable run artifacts so model-driven work can be inspected and reproduced.

## Architecture Map

- `pico.cli` assembles configuration, provider clients, workspace context, and runtime objects.
- `pico.runtime.Pico` owns the interactive control surface and request lifecycle.
- `pico.context_manager` builds bounded prompt context from prefix, memory, history, and the current request.
- `pico.tools` defines the explicit tool allowlist exposed to the runtime.
- `pico.run_store` writes per-run artifacts for replay, audit, and debugging.

For the runtime overview, see `docs/architecture/agent-harness-v1-overview.md`.

## Benchmark Evidence

The benchmark suite is split into four layers so different failure modes are measured independently:

- fixed harness regression
- context ablation
- working-memory ablation
- recovery / resume ablation

Reproducible benchmark artifacts are archived under `benchmarks/results/main-benchmark-repro-2026-06-07/`, including a generated core report and raw JSON outputs for each layer.

## Run Artifacts

Each run writes the same three primary artifacts:

- `.pico/runs/<run_id>/task_state.json`
- `.pico/runs/<run_id>/trace.jsonl`
- `.pico/runs/<run_id>/report.json`
