# Source Summary (AgentFlow)

## Paper & project context
- Project: AgentFlow - In-the-Flow Agentic System Optimization
- Claim chính: tối ưu planner trong hệ multi-agent/tool workflow bằng Flow-GRPO
- News trong README gốc: accepted ICLR 2026 (2026-01-26)

## Top-level structure (repo gốc)
- `agentflow/agentflow/`: core inference stack
- `train/`: rollout script, training launch scripts, config YAML
- `agentflow/verl/`: bridge sang RL trainer (`verl`) + Ray
- `data/`: script chuẩn bị train/val parquet
- `quick_start.py`: demo inference end-to-end

## Thành phần quan trọng
- Inference orchestration:
  - `agentflow/agentflow/solver.py`
  - `agentflow/agentflow/models/planner.py`
  - `agentflow/agentflow/models/executor.py`
  - `agentflow/agentflow/models/verifier.py`
  - `agentflow/agentflow/models/memory.py`
- Training orchestration:
  - `train/rollout.py`
  - `train/train_agent.py`
  - `train/config.yaml`
  - `agentflow/verl/entrypoint.py`
  - `agentflow/verl/trainer.py`

## Key design idea
- Thay vì train single model làm mọi thứ, AgentFlow phân module:
  1. Planner: phân tích query + chọn tool bước kế
  2. Executor: sinh command + thực thi tool
  3. Verifier: quyết định STOP/CONTINUE
  4. Generator: tổng hợp final/direct answer
- Training nhắm vào model “trainable” cho planner chính (và có thể cấu hình thêm), trong khi các thành phần khác có thể giữ fixed model.
