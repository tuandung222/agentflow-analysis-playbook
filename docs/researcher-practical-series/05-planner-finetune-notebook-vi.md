# 05. Finetune Planner trong Notebook: Protocol Thực Chiến

## 1) Setup nghiên cứu
- Trainable: Planner main.
- Fixed: Worker/Verifier/Generator.
- Eval: planning-aware metrics + final correctness.

## 2) Notebook chính
- `04_planner_finetune_eval.ipynb`

Notebook gồm:
1. Build experiment matrix.
2. Sinh command chạy training.
3. Thu và parse rollout artifacts.
4. Tổng hợp metric table.
5. So sánh checkpoint theo planning metrics.

## 3) Planning metrics đề xuất
- Final accuracy.
- Avg steps to solve.
- Invalid tool action rate.
- Verifier stop quality proxy.
- Cost per solved sample.

## 4) Benchmark groups gợi ý
- Retrieval planning.
- Math/tool program planning.
- Mixed-domain generalization split.

## 5) Mẹo để finetune ổn định hơn
- Freeze verifier/executor trước.
- Dùng reward shaping nhẹ cho efficiency.
- Log seed và endpoint version bắt buộc.
- So sánh bằng CI, không chỉ mean.
