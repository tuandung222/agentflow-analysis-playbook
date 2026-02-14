# Extension Guide

## A) Add a new tool
1. Create `agentflow/agentflow/tools/<tool_name>/`
2. Implement a `...Tool` class in `tool.py` with complete metadata
3. Define a clear `TOOL_NAME` so Initializer mapping works reliably
4. Run tool-level tests before enabling in training

## B) Add a new dataset
1. Normalize to parquet format expected by the training loader
2. Include fields such as `question`, `result`, and `extra_info`
3. Run sanity checks (nulls, length limits, duplicates)

## C) Improve reward design
- Current setup is mostly correctness-based binary reward
- Extension directions:
  - efficiency reward (fewer steps)
  - tool reliability reward
  - intermediate-goal shaping reward

## D) Tune training config
- Main file: `train/config.yaml`
- Parameter groups to prioritize:
  - throughput: `data.train_batch_size`, `rollout.n`
  - stability: learning rate, `kl_loss_coef`, clip ratios
  - latency/memory: `max_prompt_length`, `max_response_length`, `gpu_memory_utilization`

## E) Production hardening checklist
- Per-tool timeout policy
- Retry/backoff for API failures
- Guardrails for generated command execution
- Step-level tracing + structured observability metrics
