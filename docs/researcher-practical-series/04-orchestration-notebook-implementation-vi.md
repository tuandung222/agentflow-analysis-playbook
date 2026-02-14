# 04. Hiện thực Orchestration Flow trong Jupyter Notebook

## 1) Mục tiêu
- Tự build lại loop orchestration trong notebook để nhìn rõ dữ liệu chảy.
- Tạo trace table từng step để debug trajectory.

## 2) Loop chuẩn cần có
1. `analyze_query`
2. `generate_next_step`
3. `generate_tool_command`
4. `execute_tool_command`
5. `memory.add_action`
6. `verificate_context`
7. break hoặc tiếp tục
8. `generate_direct_output`

## 3) Artifacts cần log
- step_id,
- context,
- sub_goal,
- tool_name,
- command,
- tool_result,
- verifier decision,
- cumulative latency.

## 4) Notebook bạn cần chạy
- `03_orchestration_rebuild.ipynb`

Notebook này chứa:
- mock classes để chạy offline,
- state machine loop,
- visualization dạng table + step trace.

## 5) Bài tập mở rộng
- Thêm retry policy cho worker.
- Thêm verifier confidence threshold.
- Thử hard stop theo cost budget thay vì step budget.
