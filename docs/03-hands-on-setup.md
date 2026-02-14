# Hands-on Setup

## 1) Clone source gốc
```bash
git clone https://github.com/lupantech/AgentFlow.git
cd AgentFlow
```

## 2) Cài môi trường
```bash
bash setup.sh
source .venv/bin/activate
cp agentflow/.env.template agentflow/.env
```

## 3) Cấu hình key
Điền tối thiểu trong `agentflow/.env`:
- `OPENAI_API_KEY` (thường cần cho judge/tool)
- `GOOGLE_API_KEY` (nếu dùng Google search tool)

## 4) Smoke test
```bash
bash agentflow/agentflow/tools/test_all_tools.sh
python agentflow/scripts/test_llm_engine.py
python quick_start.py
```

## 5) Chạy training nhanh (thực tế)
Terminal A:
```bash
bash train/serve_with_logs.sh
```
Terminal B:
```bash
bash train/train_with_logs.sh
```

## 6) Thiết lập cho người mới (khuyến nghị)
- Giảm `N_GPUS`, `data.train_batch_size`, `actor_rollout_ref.rollout.n`
- Bắt đầu với `TOOL_STEPS=2` hoặc `3`
- Chạy 1-2 epoch để xác nhận pipeline trước khi scale

## 7) Kiểm tra artifact
- Rollout JSON thường ở `rollout_data/...`
- Log serving/training ở `task_logs/...`
- Dùng các file này để debug reward mismatch và tool failure
