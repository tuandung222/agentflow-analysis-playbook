# System Architecture

## End-to-end inference loop
Trong `solver.solve()`:
1. `planner.analyze_query()`
2. Loop theo step (`max_steps`, `max_time`):
   - `planner.generate_next_step()` -> context + sub_goal + tool
   - `executor.generate_tool_command()`
   - `executor.execute_tool_command()`
   - `memory.add_action()`
   - `verifier.verificate_context()` -> STOP/CONTINUE
3. Kết thúc loop:
   - `planner.generate_final_output()` (detailed)
   - `planner.generate_direct_output()` (concise answer)

## Vai trò từng module
- Planner:
  - Dùng LLM để phân tích mục tiêu và chọn tool theo từng bước
  - Có tách `llm_engine` và `llm_engine_fixed` để ổn định một số bước
- Executor:
  - Convert sub-goal thành code gọi `tool.execute(...)`
  - Parse command, chạy có timeout, lưu execution result
- Verifier:
  - Đánh giá memory đã đủ trả lời chưa
  - Output signal STOP/CONTINUE
- Memory:
  - Lưu action history (tool, sub-goal, command, result)

## Tooling subsystem
- `Initializer` quét `tools/**/tool.py`, nạp metadata + cache instance tool
- Có mapping short/long tool name để tăng độ robust khi planner output tên tool
- Có hỗ trợ parallel loading tool để giảm startup latency

## Lưu ý kỹ thuật
- Nhiều bước dùng prompt-template cứng trong code => behavior rất phụ thuộc prompt quality
- `exec` command sinh từ LLM trong Executor -> linh hoạt cao, nhưng rủi ro runtime/safety cao
- Cần quản lý timeout và sandbox tool chặt hơn nếu production hóa
