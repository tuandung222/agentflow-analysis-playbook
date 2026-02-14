# Concepts Primer

## RL cho LLM (ngắn gọn)
- SFT: dạy model bắt chước dữ liệu mẫu
- RL phase: tối ưu policy theo reward mục tiêu
- PPO/GRPO family: cập nhật policy có ràng buộc để không lệch quá xa

## Vì sao AgentFlow khác kiểu single-model tool use
- Single-model: một LLM vừa nghĩ vừa gọi tool => context dài, khó ổn định
- AgentFlow: tách module chuyên trách => dễ kiểm soát, dễ thay thế từng phần

## Planner / Executor / Verifier pattern
- Planner: chọn hành động
- Executor: biến action thành tương tác công cụ
- Verifier: kiểm tra đã đủ bằng chứng chưa
- Pattern này gần với control loop trong systems engineering

## Reward trong bài toán orchestration
- Reward không chỉ phụ thuộc answer cuối
- Chất lượng trajectory (tool chọn đúng, ít bước thừa, kết quả nhất quán) rất quan trọng
- Sparse reward (0/1) dễ gây variance cao; practical system thường cần shaping hoặc proxy metrics

## Độ khó cốt lõi
- Credit assignment qua chuỗi bước dài
- Tool nondeterminism (web/API đổi theo thời điểm)
- Latency & cost khi training online với tool thật
