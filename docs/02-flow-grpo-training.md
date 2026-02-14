# Flow-GRPO Training in AgentFlow

## Training stack
- Launcher: `train/train_agent.py`
- Config: `train/config.yaml`
- Rollout worker: `train/rollout.py`
- PPO core: `agentflow/verl/entrypoint.py` + `agentflow/verl/trainer.py`
- RL backend: `verl` + Ray + vLLM

## High-level dataflow
1. Parse YAML and export env vars (`train/train_agent.py`)
2. Run `python -m agentflow.verl ...`
3. In `entrypoint.py`:
   - initialize Ray
   - load tokenizer/processor
   - construct Actor/Critic/(Ref/RewardModel) workers
4. `AgentFlowTrainer` + `AgentModeDaemon`:
   - dispatch task batches to AgentFlow server
   - collect rollout triplets
   - convert rollouts to token-level PPO training batches
5. PPO step:
   - old_log_prob
   - ref_log_prob (if KL is enabled)
   - critic values
   - advantage (`grpo` per config)
   - actor/critic optimization

## Role of `train/rollout.py`
- Wraps solver execution into train/validation rollouts
- Per task:
  - appends `<answer>...</answer>` output-format constraint
  - runs solver
  - extracts answer
  - computes reward (currently via LLM-based judge)
  - writes rollout JSON artifacts
- Uses `FileLock` to synchronize step folder assignment across async workers

## Important beginner takeaways
- “Flow-GRPO” in this repo is expressed through `algorithm.adv_estimator: grpo` and `verl` PPO internals
- Rollout quality is highly sensitive to:
  - enabled tools + tool engines
  - model split (trainable vs fixed engines)
  - timeout, max steps, max response length
- Reward is sparse binary (0/1), which can increase variance and instability

## Common bottlenecks
- OOM from large `max_prompt_length` and aggressive vLLM memory settings
- Ray cluster instability or async queue overload
- Reward noise from judge inconsistency
- High API cost if fixed components and judge use commercial models
