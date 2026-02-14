# 01. Thiết kế Sub-Agent và Interface Contracts

Tài liệu này mô tả **thiết kế chức năng** và **contract input/output** của từng sub-agent trong AgentFlow, bám sát code hiện tại.

## 1) Kiến trúc role-level

- `Planner`: phân tích query, chọn bước tiếp theo, và tổng hợp output cuối.
- `Worker` (trong code là `Executor`): sinh lệnh gọi tool + thực thi.
- `Verifier`: quyết định dừng (`STOP`) hay tiếp tục (`CONTINUE`).
- `Generator`: logic tạo câu trả lời cuối; trong code hiện tại nằm trong `Planner`.

## 2) Planner contract

Nguồn tham chiếu: `agentflow/agentflow/models/planner.py`.

### Method-level I/O

1. `generate_base_response(question, image, max_tokens) -> str`
- Input:
  - `question: str`
  - `image: Optional[str]`
  - `max_tokens: int`
- Output:
  - `str` (raw base response, không tool loop)

2. `analyze_query(question, image) -> QueryAnalysis | str`
- Input:
  - `question`, `image`
  - metadata tool và available tools đã inject từ constructor
- Output logic:
  - schema `QueryAnalysis` (concise_summary, required_skills, relevant_tools, additional_considerations)
  - downstream thường convert thành `str`

3. `generate_next_step(question, image, query_analysis, memory, step_count, max_step_count, json_data) -> NextStep | str`
- Input:
  - `query_analysis` + `memory.get_actions()`
  - metadata tool
- Output:
  - schema `NextStep` (`justification`, `context`, `sub_goal`, `tool_name`) hoặc string parseable

4. `extract_context_subgoal_and_tool(response) -> Tuple[str, str, str]`
- Input:
  - output từ `generate_next_step`
- Output:
  - `context`, `sub_goal`, `tool_name` đã normalize theo danh sách tool hợp lệ

5. `generate_final_output(question, image, memory) -> str`
- Input:
  - toàn bộ action history
- Output:
  - detailed solution

6. `generate_direct_output(question, image, memory) -> str`
- Input:
  - toàn bộ action history
- Output:
  - concise answer

### Kỳ vọng định dạng đầu ra của Planner

Ở step planning, Planner phải xuất ra tối thiểu:
- `Context`: thông tin cần cho tool chạy đúng
- `Sub-Goal`: mục tiêu hành động cụ thể
- `Tool Name`: tên tool khớp chính xác với toolbox

Nếu Planner sai ở 1 trong 3 trường này, loop dễ fail ở Executor.

## 3) Worker/Executor contract

Nguồn tham chiếu: `agentflow/agentflow/models/executor.py`.

### Method-level I/O

1. `set_query_cache_dir(query_cache_dir)`
- Input:
  - path cache cho query hiện tại
- Output:
  - tạo thư mục output cho artifacts của tool execution

2. `generate_tool_command(question, image, context, sub_goal, tool_name, tool_metadata, step_count, json_data) -> ToolCommand | str`
- Input:
  - semantic plan từ Planner (`context`, `sub_goal`, `tool_name`)
  - `tool_metadata`
- Output:
  - `ToolCommand` gồm `analysis`, `explanation`, `command`
  - `command` kỳ vọng chứa `execution = tool.execute(...)`

3. `extract_explanation_and_command(response) -> Tuple[str, str, str]`
- Input:
  - response từ command generator
- Output:
  - `analysis`, `explanation`, `command` đã normalize

4. `execute_tool_command(tool_name, command) -> Any`
- Input:
  - `tool_name`
  - code string (`command`)
- Output:
  - list kết quả execution hoặc lỗi
- Lưu ý:
  - command split thành block
  - mỗi block chạy timeout-protected

### Kỳ vọng đầu vào của Worker

Worker không tự suy luận mục tiêu toàn cục. Nó cần Planner cấp:
- context đủ dùng,
- sub-goal rõ,
- tool name hợp lệ.

Nếu context thiếu biến/file path, command generation sẽ dễ hallucinate.

## 4) Verifier contract

Nguồn tham chiếu: `agentflow/agentflow/models/verifier.py`.

### Method-level I/O

1. `verificate_context(question, image, query_analysis, memory, step_count, json_data) -> MemoryVerification | str`
- Input:
  - query ban đầu
  - query_analysis
  - current `memory.get_actions()`
  - tool metadata
- Output:
  - `MemoryVerification` gồm:
    - `analysis: str`
    - `stop_signal: bool`

2. `extract_conclusion(response) -> Tuple[str, str]`
- Input:
  - response verifier
- Output:
  - `(analysis, conclusion)` với `conclusion in {STOP, CONTINUE}`

### Kỳ vọng chức năng

Verifier là gating policy:
- đánh giá memory đã đủ để trả lời chưa,
- kiểm soát compute budget và quality.

## 5) Memory contract

Nguồn tham chiếu: `agentflow/agentflow/models/memory.py`.

### Method-level I/O

- `add_action(step_count, tool_name, sub_goal, command, result)`
- `get_actions() -> Dict[str, Dict[str, Any]]`

Memory là structured trace dùng cho:
- Planner step t+1,
- Verifier stop decision,
- Generator final synthesis.

## 6) Solver-level orchestration contract

Nguồn: `agentflow/agentflow/solver.py`.

Solver đảm bảo:
- sequencing chuẩn giữa sub-agents,
- loop boundary (`max_steps`, `max_time`),
- ghi `json_data` để audit toàn bộ pipeline.

## 7) Cấu hình “chỉ tune Planner, cố định sub-agent khác”

Trong `construct_solver`, `model_engine` map theo thứ tự:
- `[planner_main, planner_fixed, verifier, executor]`

Để tune planner main và giữ role còn lại fixed:
- `planner_main = trainable endpoint`
- `planner_fixed = fixed endpoint/model`
- `verifier = fixed endpoint/model`
- `executor = fixed endpoint/model`

Tư duy nghiên cứu:
- planner học decision policy,
- các role còn lại đóng vai trò environment/stationary evaluators (gần đúng).
