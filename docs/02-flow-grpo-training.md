# Flow-GRPO Training in AgentFlow

## Training stack
- Launcher: `train/train_agent.py`
- Config: `train/config.yaml`
- Rollout worker: `train/rollout.py`
- PPO core: `agentflow/verl/entrypoint.py` + `agentflow/verl/trainer.py`
- Backend RL framework: `verl` + Ray + vLLM

## Dataflow tổng quát
1. Parse YAML + export env vars (`train/train_agent.py`)
2. Run `python -m agentflow.verl ...`
3. Trong `entrypoint.py`:
   - init Ray
   - load tokenizer/processor
   - tạo worker group Actor/Critic/(Ref/RewardModel)
4. `AgentFlowTrainer` gọi `AgentModeDaemon` để:
   - gửi batch task tới AgentFlow server
   - thu rollout triplets
   - biến về batch token-level cho PPO update
5. PPO step:
   - old_log_prob
   - ref_log_prob (nếu dùng KL)
   - value (critic)
   - advantage (`grpo` theo config)
   - actor/critic optimization

## Vai trò của `train/rollout.py`
- Bọc solver thành agent rollout cho train/validation
- Với mỗi task:
  - append format yêu cầu `<answer>...</answer>`
  - chạy solver
  - extract answer
  - chấm reward (đang dùng LLM judge qua `compute_score`)
  - lưu JSON rollout artifact
- Dùng `FileLock` để sync step folder giữa nhiều worker async

## Những điểm cần hiểu rõ cho người mới
- “Flow-GRPO” ở repo được hiện thực qua cấu hình `algorithm.adv_estimator: grpo` và hạ tầng PPO của `verl`
- Rollout quality phụ thuộc mạnh vào:
  - tool set + tool engine
  - model_engine split (trainable vs fixed)
  - timeout, max steps, max response length
- Reward hiện tại là sparse binary (0/1) theo judge => variance cao, cần nhiều rollout/ổn định hạ tầng

## Bottlenecks thường gặp
- OOM do `max_prompt_length` lớn + vLLM memory utilization cao
- Deadlock/lag do Ray cluster hoặc async queue overload
- Reward noise khi judge không ổn định (prompt judge chưa đủ chặt)
- Cost API cao nếu dùng model thương mại cho judge/tool/planner fixed
