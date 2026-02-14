# Source Summary (AgentFlow)

## Paper and project context
- Project: AgentFlow - In-the-Flow Agentic System Optimization
- Core claim: optimize the planner inside a multi-agent/tool workflow with Flow-GRPO
- The upstream README news section reports ICLR 2026 acceptance (2026-01-26)

## Top-level structure (upstream repo)
- `agentflow/agentflow/`: core inference stack
- `train/`: rollout script, training launch scripts, config YAML
- `agentflow/verl/`: bridge to RL trainer (`verl`) + Ray
- `data/`: scripts to prepare train/validation parquet files
- `quick_start.py`: end-to-end inference demo

## Critical components
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
- Instead of training one model to do everything, AgentFlow splits responsibilities:
  1. Planner: query analysis + next-tool selection
  2. Executor: command generation + tool execution
  3. Verifier: STOP/CONTINUE decision
  4. Generator: final/direct response synthesis
- Training primarily targets the “trainable” model in the planner path, while other components can remain fixed.
