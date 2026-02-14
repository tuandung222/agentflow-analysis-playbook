# Code Deep Dive

## Inference path
- Minimal entry: `quick_start.py`
- Main wiring: `agentflow/agentflow/solver.py:construct_solver`

### `construct_solver` wiring
- Initializer: loads tools, metadata, and cached tool instances
- Planner: includes both `llm_engine` (main) and `llm_engine_fixed`
- Verifier: dedicated engine for STOP/CONTINUE decisions
- Executor: consumes `tool_instances_cache` to avoid repeated tool initialization
- Memory: stores action-level state across steps

### `Solver.solve` state machine
- `analyze_query` to set high-level direction
- Per-step loop:
  - `generate_next_step`
  - `extract_context_subgoal_and_tool`
  - `generate_tool_command`
  - `execute_tool_command`
  - `add_action`
  - `verificate_context` + `extract_conclusion`
- Exit when STOP is reached or `max_steps/max_time` limit is hit

## Training path
- Driver: `train/train_agent.py`
- Rollout implementation: `train/rollout.py`
- RL loop on `verl`: `agentflow/verl/entrypoint.py`, `agentflow/verl/trainer.py`

### AgentFlow <-> verl integration points
- `Rollout` (`LitAgent` subclass) executes real queries and emits rollout artifacts
- `AgentFlowTrainer` converts rollouts into token-level PPO update batches
- `AgentDataset` injects `fake_ids` to satisfy DataProto requirements

## Critical dependencies
- Ray: worker scheduling and async rollout infrastructure
- vLLM: serves the trainable model for rollout actors
- External APIs: OpenAI/Google/etc. for tools, judging, and fixed components

## Technical risks
1. Generated command execution via `exec`:
   - runtime and safety risks if not sandboxed
2. Binary reward design:
   - high reward noise and variance
3. Multi-component non-determinism:
   - difficult strict reproducibility
4. Context growth:
   - memory and tool outputs can inflate prompts rapidly

## Production hardening directions
- Introduce a constrained command DSL instead of raw `exec`
- Add deterministic rollout replay harnesses
- Decompose reward (correctness + efficiency + tool trust)
- Add policy guardrails before sensitive tool calls
