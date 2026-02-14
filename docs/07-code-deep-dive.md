# Code Deep Dive

## Inference path
- Entry đơn giản: `quick_start.py`
- Construct stack: `agentflow/agentflow/solver.py:construct_solver`

### `construct_solver` wiring
- Initializer: load tools + metadata + cache instance
- Planner: gồm `llm_engine` (main) + `llm_engine_fixed`
- Verifier: engine riêng cho STOP/CONTINUE
- Executor: nhận `tool_instances_cache` để tránh khởi tạo lại tool
- Memory: trạng thái theo từng action step

### `Solver.solve` state machine
- `analyze_query` -> tạo plan cấp cao
- Loop per step:
  - `generate_next_step`
  - `extract_context_subgoal_and_tool`
  - `generate_tool_command`
  - `execute_tool_command`
  - `add_action`
  - `verificate_context` + `extract_conclusion`
- Exit condition: `STOP` hoặc chạm giới hạn `max_steps/max_time`

## Training path
- Driver: `train/train_agent.py`
- Rollout implementation: `train/rollout.py`
- RL loop on verl: `agentflow/verl/entrypoint.py`, `agentflow/verl/trainer.py`

### Điểm giao nhau AgentFlow <-> verl
- `Rollout` (kế thừa `LitAgent`) thực thi query thật và ghi rollout artifacts
- `AgentFlowTrainer` thu rollout để build token-level batch cho PPO update
- `AgentDataset` thêm `fake_ids` để tương thích DataProto

## Critical dependencies
- Ray: scheduling worker và async rollout infra
- vLLM: serving model trainable cho rollout actor
- External APIs: OpenAI/Google/... cho tool/judge/fixed components

## Technical risks
1. Generated command via `exec`:
   - rủi ro runtime exception/safety
2. Binary reward judge:
   - reward noise cao
3. Multi-component non-determinism:
   - khó reproducibility tuyệt đối
4. Context explosion:
   - memory + tool outputs kéo dài prompt

## Nếu cần harden production
- Introduce strict DSL thay vì `exec` trực tiếp
- Add deterministic replay harness cho rollout
- Add richer reward decomposition (correctness + efficiency + tool-trust)
- Add guardrails/filters trước khi gọi tool nhạy cảm
