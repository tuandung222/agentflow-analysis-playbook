# Hands-on Setup

## 1) Clone upstream source
```bash
git clone https://github.com/lupantech/AgentFlow.git
cd AgentFlow
```

## 2) Install environment
```bash
bash setup.sh
source .venv/bin/activate
cp agentflow/.env.template agentflow/.env
```

## 3) Configure API keys
Set at least these in `agentflow/.env`:
- `OPENAI_API_KEY` (commonly needed for judge/tooling)
- `GOOGLE_API_KEY` (if using Google search tool)

## 4) Smoke tests
```bash
bash agentflow/agentflow/tools/test_all_tools.sh
python agentflow/scripts/test_llm_engine.py
python quick_start.py
```

## 5) Run practical training
Terminal A:
```bash
bash train/serve_with_logs.sh
```
Terminal B:
```bash
bash train/train_with_logs.sh
```

## 6) Beginner-friendly config strategy
- Lower `N_GPUS`, `data.train_batch_size`, `actor_rollout_ref.rollout.n`
- Start with `TOOL_STEPS=2` or `3`
- Validate 1-2 short epochs before scaling up

## 7) Check artifacts
- Rollout JSON artifacts: typically in `rollout_data/...`
- Serving/training logs: `task_logs/...`
- Use these files to debug reward mismatch and tool failures
