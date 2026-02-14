# 04. Protocol Tuning Planner (Fix Worker/Verifier/Generator)

Tài liệu này tập trung vào setup bạn yêu cầu:
- Tune `Planner` cho bài toán planning.
- `Worker(Executor)`, `Verifier`, `Generator` dùng model fixed qua inference endpoint có sẵn.

## 1) Mục tiêu tối ưu

Bạn đang tối ưu policy ở decision layer:
- chọn hành động kế tiếp,
- chọn tool đúng,
- đặt sub-goal đúng,
- giảm trajectory thừa.

Đây là objective “planning policy optimization”, không chỉ text generation optimization.

## 2) Cấu hình role routing

Trong AgentFlow, `model_engine` map theo:
- `[planner_main, planner_fixed, verifier, executor]`

Cấu hình cho research setup này:
- `planner_main = trainable`
- `planner_fixed = fixed_endpoint_model`
- `verifier = fixed_endpoint_model`
- `executor = fixed_endpoint_model`

Ý nghĩa:
- Planner main policy được update qua RL.
- Các role còn lại tạo “semi-stationary environment”.

## 3) Pseudocode training protocol

```text
for each training step:
    sample minibatch of tasks

    for each task:
        run solver rollout with role routing:
            planner_main -> trainable model
            planner_fixed/verifier/executor -> fixed model endpoint

        collect triplets (prompt_ids, response_ids) and final reward

    convert rollouts to PPO batch:
        pad/truncate
        build attention masks
        build token_level_scores (reward at final token)

    run PPO/GRPO update on trainable policy

    log planning-specific metrics and reliability metrics
```

## 4) Cách giữ Worker/Verifier/Generator cố định đúng nghĩa

1. Không set các role đó thành `trainable` trong `MODEL_ENGINE`.
2. Pin endpoint/model version cho fixed roles (không thay đổi giữa các run).
3. Pin sampling params cho fixed roles (`temperature`, `top_p`) để giảm non-stationarity.
4. Cache tool/environment behavior nếu có thể (đặc biệt search APIs).

## 5) Reward design cho planner tuning

Reward hiện tại trong code là binary correctness (0/1). Đủ để bắt đầu, nhưng cho planning research nên mở rộng:

`R_total = w1 * R_answer + w2 * R_efficiency + w3 * R_tool_valid + w4 * R_stop_quality`

Trong đó:
- `R_answer`: đúng/sai câu trả lời cuối.
- `R_efficiency`: thưởng trajectory ngắn và hợp lý.
- `R_tool_valid`: phạt command lỗi/timeout/tool misuse.
- `R_stop_quality`: thưởng dừng đúng lúc (không thiếu/không thừa bước).

## 6) Planner-focused metrics (nên log bắt buộc)

1. `Planner Action Validity`:
- tỉ lệ step mà tool_name hợp lệ và parse thành công.

2. `Planner Tool Utility`:
- tỉ lệ step mà tool call đóng góp thông tin hữu ích vào câu trả lời cuối.

3. `Trajectory Efficiency`:
- số bước trung bình để đạt correctness.

4. `Decision Stability`:
- variance của tool selection trên cùng prompt với nhiều seed.

5. `Outcome`:
- final answer accuracy/pass@1.

## 7) Pitfalls lớn khi chỉ tune Planner

1. Reward leakage từ Generator style
- nếu judge phụ thuộc style output, planner có thể học hành vi gián tiếp không mong muốn.

2. Verifier bias
- verifier fixed quá “dễ STOP” hoặc quá “khó STOP” sẽ bias learning signal của planner.

3. Environment drift
- tool APIs/web results thay đổi theo thời gian làm nhiễu RL signal.

4. Credit assignment thô
- reward đặt cuối trajectory gây khó học ở decision sớm.

## 8) Cách giảm nhiễu thí nghiệm (paper-grade)

1. Fixed seed + fixed eval set + fixed endpoint versions.
2. Report CI (confidence interval), không chỉ mean.
3. Ablation theo từng thành phần reward.
4. Compare với baselines:
- no-RL planner,
- RL planner + fixed verifier,
- RL planner + tuned verifier (optional expansion).

## 9) Checklist triển khai nhanh

- [ ] `MODEL_ENGINE` đúng role routing.
- [ ] Endpoint fixed roles không đổi version.
- [ ] Logging đầy đủ step-level outputs (`json_data`).
- [ ] Metrics planner-specific được tính riêng.
- [ ] Đánh giá cost/latency song song với accuracy.
