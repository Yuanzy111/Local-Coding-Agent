# Pico Benchmark Reproducibility Guide

本目录保存的是 Pico 本地 agent harness 的一组可复核 benchmark 结果。它们用于说明当前 main 分支在固定回归、上下文治理、结构化记忆和 checkpoint / resume 恢复上的表现，不代表线上业务统计。

## 目录内容

| 文件 | 用途 |
| --- | --- |
| `harness-regression-v2.json` | 固定 harness regression 任务结果 |
| `context-ablation-v2.json` | 长上下文治理对照实验 |
| `memory-ablation-v2.json` | 结构化记忆对照实验 |
| `recovery-ablation-v2.json` | checkpoint / resume 恢复实验 |
| `pico-benchmark-core-report.md` | 自动生成的 benchmark 汇总 |

本归档只提交可复核的 JSON 和 Markdown 结果，不提交临时 workspace 副本。每题的摘要、verifier、状态和运行工件路径已写入 `harness-regression-v2.json`。

## 复现命令

在仓库根目录执行：

```bash
uv run python - <<'PY'
from pathlib import Path
from pico.evaluation.evaluator import run_harness_regression_v2
from pico.evaluation.metrics import (
    run_context_ablation_v2,
    run_memory_ablation_v2,
    run_recovery_ablation_v2,
    write_benchmark_core_report,
)

out = Path("benchmarks/results/main-benchmark-repro-2026-06-07")
run_harness_regression_v2(
    benchmark_path=Path("benchmarks/coding_tasks.json"),
    artifact_path=out / "harness-regression-v2.json",
    workspace_root=Path("/tmp/pico-main-benchmark-workspaces"),
)
run_context_ablation_v2(out / "context-ablation-v2.json", repetitions=5)
run_memory_ablation_v2(out / "memory-ablation-v2.json", repetitions=5)
run_recovery_ablation_v2(out / "recovery-ablation-v2.json", repetitions=3)
write_benchmark_core_report(
    report_path=out / "pico-benchmark-core-report.md",
    harness_artifact_path=out / "harness-regression-v2.json",
    context_artifact_path=out / "context-ablation-v2.json",
    memory_artifact_path=out / "memory-ablation-v2.json",
    recovery_artifact_path=out / "recovery-ablation-v2.json",
)
PY
```

## 指标解释

### 1. Harness Regression

这组任务验证 runtime 合同是否稳定，包括工具调用、路径约束、运行工件写出、部分成功识别和 verifier 流程。当前归档结果为：

| 指标 | 值 |
| --- | --- |
| 固定任务数 | 12 |
| 通过率 | 100% |
| 预算内完成率 | 100% |
| verifier 通过率 | 100% |

### 2. Context Ablation

`run_context_ablation_v2()` 构造 12 组固定上下文压力配置，对比开启上下文治理前后的 prompt 长度，并检查当前请求是否被保留。当前归档结果为：

| 指标 | 值 |
| --- | --- |
| 配置数 | 12 |
| 平均压缩前 prompt 长度 | 6994.33 |
| 平均压缩后 prompt 长度 | 5575.67 |
| 平均压缩率 | 16.36% |
| 最高压缩率 | 33.59% |
| 当前请求保留率 | 100% |

这些数字会随着 prompt 模板和上下文段落的微调而略有波动，但实验目标保持不变：在不裁坏当前请求的前提下缩短 prompt。

### 3. Working Memory Ablation

`run_memory_ablation_v2(repetitions=5)` 构造 12 个 memory dependency 任务，对比 `memory_off`、`memory_irrelevant` 和 `memory_on` 三个 variant。当前归档结果为：

| 指标 | 值 |
| --- | --- |
| 任务数 | 12 |
| 每个 variant 运行数 | 60 |
| memory off 重复读 | 60 |
| memory on 重复读 | 0 |
| memory off 平均工具步数 | 1.00 |
| memory on 平均工具步数 | 0.00 |
| memory on 正确率 | 100% |
| memory 命中率 | 100% |

这里的重点是：当相关事实已经进入结构化记忆后，follow-up 阶段不再需要重复读文件确认同一事实。

### 4. Recovery / Resume Ablation

`run_recovery_ablation_v2(repetitions=3)` 构造 10 个恢复相关任务，覆盖 checkpoint resume、partial stale、workspace mismatch、schema mismatch 和 partial success recovery 等场景。当前归档结果为：

| 指标 | 值 |
| --- | --- |
| 恢复任务数 | 10 |
| 每个 variant 运行数 | 30 |
| resume enabled 成功率 | 90% |
| stale reanchor 率 | 100% |
| workspace 漂移识别率 | 100% |
| false accept 率 | 0% |

这里的 `workspace_drift_detection_rate` 和 `resume_false_accept_rate` 用于证明恢复边界没有错误放宽。

### 5. Scope Notes

- 这些 benchmark 用来验证本地 harness 设计，不等价于 provider 能力排行榜。
- Harness regression、context、memory 和 recovery 是拆层测量的，目的是把不同模块收益和失败模式分开看。
- 所有实验都依赖仓库内固定任务、固定 verifier 和运行工件，因此可以复核，不只看模型最终自然语言回答。
