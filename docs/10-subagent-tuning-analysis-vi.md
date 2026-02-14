# Phân tích tuning sub-agent trong AgentFlow + `verl` (kèm mã giả)

Tài liệu này trả lời các câu hỏi:
- Mã nguồn này vì sao tune được “sub-agent” trong hệ multi-agent?
- Planner/Verifier/Executor tune thế nào?
- Có tune đồng thời được không?
- Dữ liệu tuning là gì?

## 1) Vì sao có thể tune ở mức sub-agent?

Điểm mấu chốt là **tách vai trò bằng prompt + call-site riêng**:
- Planner call qua `planner.generate_next_step(...)`.
- Verifier call qua `verifier.verificate_context(...)`.
- Executor call qua `executor.generate_tool_command(...)`.
- Generator call qua `planner.generate_final_output(...)` / `generate_direct_output(...)`.

Mỗi role là một invocation riêng, với prompt schema riêng, nên có thể “bật trainable” cho từng role bằng config model routing.

## 2) Cơ chế isolate chức năng agent trong code

## 2.1 Isolate theo class/method
- Planner, Verifier, Executor là class riêng.
- Generator là vai trò riêng nhưng nằm trong Planner methods.

## 2.2 Isolate theo model routing (`model_engine`)

Trong `construct_solver(...)`, mapping engine được tách theo role:
- `planner_main_engine = llm_engine_name if model_engine[0] == "trainable" else model_engine[0]`
- `planner_fixed_engine = llm_engine_name if model_engine[1] == "trainable" else model_engine[1]`
- `verifier_engine = llm_engine_name if model_engine[2] == "trainable" else model_engine[2]`
- `executor_engine = llm_engine_name if model_engine[3] == "trainable" else model_engine[3]`

Nghĩa là mỗi role có thể dùng:
- model trainable (served bởi vLLM/proxy của training loop), hoặc
- model fixed (gpt-4o, dashscope, ...).

## 3) Có tune đồng thời các sub-agent được không?

Ngắn gọn: **Có thể**, nhưng cần hiểu đúng bản chất hiện tại.

## 3.1 Mặc định hiện tại trong config
Mặc định repo thường để:
- Planner main = `trainable`
- Các role còn lại = fixed model

=> Thực tế chủ yếu tune planner policy.

## 3.2 Nếu muốn tune đồng thời nhiều role
Bạn có thể set nhiều vị trí trong `MODEL_ENGINE` là `"trainable"` (planner/verifier/executor/...).

Khi đó:
- các role đó cùng gọi về trainable endpoint,
- rollout sẽ chứa trace từ nhiều role trainable,
- PPO update tối ưu **một policy model dùng chung**.

Lưu ý quan trọng:
- Đây không phải multi-policy training độc lập từng role.
- Đây là **shared-parameter multi-role tuning** (nhiều vai trò, một trọng số model).

## 4) Dữ liệu tuning trong pipeline này là gì?

## 4.1 Nguồn dữ liệu gốc
- Dataset parquet (`question`, `result`, `extra_info`, ...), load bởi `AgentDataset`.

## 4.2 Dữ liệu rollout
Rollout trả về `triplets` (prompt token ids, response token ids, reward metadata) qua tracing/export.

Trong daemon:
- mỗi rollout chứa danh sách turn traces:
  - `prompt_ids`
  - `response_ids`
  - reward cuối cùng của rollout

## 4.3 Dữ liệu train batch cho PPO
`get_train_data_batch(...)` tạo:
- `prompts`, `responses`, `input_ids`, `attention_mask`, `position_ids`
- `token_level_scores`
- non-tensor metadata: `data_id_list`, `rollout_id_list`, `turn_index_list`

Đặc trưng hiện tại:
- final reward (0/1) được gán vào token cuối của mỗi transition.
- cùng reward cuối thường được broadcast cho các turn trong rollout sample.
- credit assignment là coarse, chưa có shaping reward theo từng sub-goal.

## 5) Mã giả giải thuật tuning hiện tại

## 5.1 Pseudocode cấp hệ thống
```text
function TRAIN_WITH_VERL(config):
    init Ray
    tokenizer, processor = load_from_base_model(config)

    train_dataset = AgentDataset(train.parquet)
    val_dataset   = AgentDataset(val.parquet)

    trainer = AgentFlowTrainer(..., datasets, reward_fn, ...)
    trainer.init_workers()
    trainer.fit()
```

## 5.2 Pseudocode rollout (agent side)
```text
function TRAINING_ROLLOUT(task):
    prompt = task.question + "<answer>...</answer> format hint"
    result = solver.solve(prompt)  # planner->executor->verifier loop
    answer = extract_answer(result.direct_output)
    reward = judge(question, groundtruth, answer)  # currently sparse 0/1
    save rollout json artifact
    return rollout trace + reward
```

## 5.3 Pseudocode daemon batch builder
```text
function BUILD_PPO_BATCH(completed_rollouts):
    transitions = []
    for each rollout in completed_rollouts:
        trace_list = [(prompt_ids, response_ids) for each triplet]
        final_reward = rollout.reward
        for each turn_trace in trace_list:
            padded_prompt, padded_resp = pad_and_truncate(turn_trace)
            transitions.append((padded_prompt, padded_resp, final_reward))

    batch = tensorize(transitions)
    token_level_scores = zeros_like(response_tokens)
    token_level_scores[last_token] = final_reward
    return batch
```

## 5.4 Pseudocode PPO/GRPO update
```text
function PPO_STEP(batch):
    old_log_prob = actor.compute_log_prob(batch)
    if use_ref_policy:
        ref_log_prob = ref.compute_log_prob(batch)
    if use_critic:
        values = critic.compute_values(batch)

    advantages = compute_advantage(batch, estimator="grpo")
    update_actor_critic(advantages, kl, clip, ...)
```

## 6) Tune từng sub-agent nên nghĩ thế nào?

## 6.1 Planner tuning
Mục tiêu:
- chọn đúng tool,
- decomposition hợp lý,
- giảm step dư.

Tác động khi tune:
- mạnh nhất lên chất lượng trajectory toàn cục.
- thường là role có leverage lớn nhất.

Nên theo dõi:
- tool selection accuracy,
- step count trung bình,
- tỉ lệ verifier STOP đúng lúc.

## 6.2 Verifier tuning
Mục tiêu:
- quyết định `STOP/CONTINUE` chính xác.

Rủi ro:
- quá dễ STOP -> thiếu bằng chứng.
- quá khó STOP -> tốn cost/latency, loop dài.

Nên theo dõi:
- early-stop error rate,
- unnecessary-continue rate.

## 6.3 Executor tuning
Mục tiêu:
- sinh command đúng cú pháp/đúng tham số tool,
- giảm runtime error.

Nên theo dõi:
- command parse success,
- tool execution success,
- tỉ lệ timeout/error theo tool.

## 6.4 Generator tuning (vai trò tổng hợp output)
Trong code hiện tại role này chưa tách class riêng.
Nếu để fixed model:
- output ổn định hơn, ít ảnh hưởng policy training.
Nếu để trainable:
- có thể tối ưu answer style theo reward, nhưng dễ làm reward hacking nếu judge lỏng.

## 7) Chiến lược tuning thực tế (đề xuất)

1. Bước 1: chỉ tune planner (`MODEL_ENGINE = [trainable, fixed, fixed, fixed]`).
2. Bước 2: nếu planner ổn, thử mở verifier thành trainable.
3. Bước 3: cuối cùng mới cân nhắc executor trainable.
4. Mỗi bước cần fixed seed/config baseline để so A/B rõ ràng.

Lý do:
- giảm nhiễu do nhiều role thay đổi cùng lúc,
- dễ truy nguyên lỗi credit assignment.

## 8) Trả lời trực diện các câu hỏi mơ hồ thường gặp

### “Các subagent có được tuning đồng thời không?”
Có, bằng cách set nhiều role là `trainable`. Nhưng chúng thường dùng chung một policy model đang optimize, không phải mỗi role một model độc lập.

### “Dạng dữ liệu tuning là gì?”
Không phải chỉ Q/A text. Dữ liệu training thực tế là trajectory token-level:
- prompt_ids/response_ids theo từng turn LLM call,
- reward cuối cùng gán vào token-level scores.

### “Tuning planner/verifier khác gì nhau?”
Khác ở prompt-task objective:
- planner học decision policy (chọn hành động),
- verifier học stopping policy (đánh giá đủ bằng chứng hay chưa).
Nhưng nếu cùng `trainable`, gradient đều đi vào cùng trọng số model dùng chung.

## 9) Kết luận

AgentFlow tune “sub-agent” được vì đã tách role theo call-path và model routing.
`verl` được dùng để học policy trên trajectory tokenized từ các lượt gọi đó.

Điểm cần nhớ:
- mặc định thực tế chủ yếu tune planner,
- multi-role tuning có thể làm được nhưng là shared-policy tuning,
- reward hiện tại còn coarse nên cần cẩn trọng khi mở rộng sang tune đồng thời nhiều role.
