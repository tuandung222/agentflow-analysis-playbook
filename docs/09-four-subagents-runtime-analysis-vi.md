# Phân tích runtime 4 sub-agent của AgentFlow (kèm mã giả)

Tài liệu này trả lời câu hỏi: “Given a user query, 4 specialized agent trong AgentFlow chạy như thế nào theo code hiện tại?”

## 1) 4 sub-agent trong paper vs trong code

Theo paper/repo mô tả: `Planner`, `Executor`, `Verifier`, `Generator`.

Trong code hiện tại:
- `Planner`: class riêng, chịu trách nhiệm phân tích query + chọn bước tiếp theo.
- `Executor`: class riêng, sinh command và thực thi tool.
- `Verifier`: class riêng, quyết định `STOP` hoặc `CONTINUE`.
- `Generator`: không tách class riêng; được hiện thực bằng 2 method trong `Planner`:
  - `generate_final_output()`
  - `generate_direct_output()`

Điểm này quan trọng vì “4 agents” là 4 vai trò logic; còn mức triển khai class thì Generator đang nằm trong Planner.

## 2) Dữ liệu và state chảy qua pipeline

Các state chính trong một lượt solve:
- `question`, `image_path` (input user).
- `query_analysis` (Planner tạo).
- `memory.actions` (dict các action step).
- `tool_result_i` (output từng tool call).
- `conclusion` từ Verifier (`STOP`/`CONTINUE`).
- `final_output` / `direct_output` (Generator phase).

Core orchestration nằm ở `agentflow/agentflow/solver.py`.

## 3) Luồng thực thi “Given a query”

## 3.1 Trình tự thực tế
1. `construct_solver(...)` khởi tạo:
   - `Initializer` (load tools + metadata + cache instances),
   - `Planner`, `Verifier`, `Executor`, `Memory`.
2. `solve(question)`:
   - Planner phân tích query (`analyze_query`).
   - Loop theo step:
     - Planner chọn tool + sub-goal (`generate_next_step`).
     - Executor sinh Python command (`generate_tool_command`).
     - Executor chạy tool (`execute_tool_command`).
     - Memory ghi action.
     - Verifier đọc memory để quyết định dừng/chạy tiếp.
   - Kết thúc loop:
     - Generator phase: tạo `final_output` và/hoặc `direct_output`.

## 3.2 Mã giả bám sát code
```text
function SOLVE(question, image_path=None):
    executor.set_query_cache_dir(root_cache_dir)
    json_data = {query: question, image: image_path}

    if "base" in output_types:
        json_data.base_response = planner.generate_base_response(question, image_path)
        if output_types == {"base"}:
            return json_data

    query_analysis = planner.analyze_query(question, image_path)
    json_data.query_analysis = query_analysis

    step_count = 0
    start_time = now()

    while step_count < max_steps and elapsed(start_time) < max_time:
        step_count += 1

        next_step = planner.generate_next_step(
            question, image_path, query_analysis, memory, step_count, max_steps, json_data
        )
        context, sub_goal, tool_name = planner.extract_context_subgoal_and_tool(next_step)

        if tool_name invalid:
            command = "none"
            result = "tool not found"
        else:
            tool_command = executor.generate_tool_command(
                question, image_path, context, sub_goal, tool_name, toolbox_metadata[tool_name], step_count, json_data
            )
            analysis, explanation, command = executor.extract_explanation_and_command(tool_command)
            result = executor.execute_tool_command(tool_name, command)
            json_data["tool_result_" + step_count] = result

        memory.add_action(step_count, tool_name, sub_goal, command, result)

        verify_resp = verifier.verificate_context(
            question, image_path, query_analysis, memory, step_count, json_data
        )
        _, conclusion = verifier.extract_conclusion(verify_resp)
        if conclusion == "STOP":
            break

    if "final" in output_types:
        json_data.final_output = planner.generate_final_output(question, image_path, memory)
    if "direct" in output_types:
        json_data.direct_output = planner.generate_direct_output(question, image_path, memory)

    return json_data
```

## 4) So sánh với ReAct, CodeAct, Planner+Worker

## 4.1 ReAct
ReAct (Reason + Act) thường là một policy đơn:
- cùng một model sinh chain-of-thought + action.
- observation quay lại model đó, loop tiếp.

AgentFlow khác ở:
- tách vai trò rõ: Planner chọn hành động, Executor tạo/làm action, Verifier là gating signal.
- có thể gán model khác nhau cho từng vai trò (`model_engine` split).
- explicit memory object thay vì “history trong prompt” thuần.

## 4.2 CodeAct
CodeAct thường nhấn mạnh:
- model sinh code/action script trực tiếp,
- runtime chạy code và feed output ngược lại.

AgentFlow giống CodeAct ở phần Executor:
- sinh Python command có `tool.execute(...)` và thực thi.

Nhưng AgentFlow khác:
- command generation bị đặt trong một role riêng (Executor), không phải toàn bộ policy.
- có Verifier độc lập để quyết định stop condition theo context completeness.
- planner dùng metadata tool để chọn action có chủ đích hơn.

## 4.3 Planner + Worker
Planner+Worker pattern truyền thống:
- Planner bẻ task thành subtask,
- Worker làm subtask,
- thường thiếu lớp “đánh giá đủ/chưa đủ” tách bạch.

AgentFlow có thêm Verifier như một control gate:
- giảm việc chốt sớm hoặc dừng quá muộn.
- tạo vòng lặp có điều kiện dừng thông minh hơn.

Nói ngắn: AgentFlow = Planner+Worker + Verifier + unified memory + trainable-in-the-loop orchestration.

## 5) Điểm kỹ thuật đáng chú ý

1. Tool command execution dùng `exec`:
- linh hoạt, nhưng rủi ro runtime/safety cao nếu production.

2. Verifier là “quality gate” chính:
- nếu verifier yếu, loop có thể dừng sai hoặc chạy vòng vo.

3. Generator phase hiện dùng engine fixed trong Planner:
- giúp answer formatting ổn định,
- nhưng làm cho role “Generator” chưa thực sự tách class độc lập.

4. Tool metadata và name mapping:
- giúp planner output tên tool robust hơn khi có biến thể naming.

## 6) Kết luận

Với một user query, AgentFlow không chạy kiểu “1 model tự nghĩ tự làm” như ReAct thuần.
Nó chạy như một state machine nhiều role:
- Planner quyết định,
- Executor hành động,
- Verifier kiểm chứng,
- Generator tổng hợp.

Cách tách role này là nền tảng để huấn luyện “in-the-flow” trên hệ multi-agent/tool, thay vì chỉ RL trên một chuỗi text đơn.
