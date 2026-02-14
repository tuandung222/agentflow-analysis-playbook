# 02. Planner Deep Dive: Lôi Planner Ra Vọc

## 1) Planner học cái gì trong hệ này?
Planner học policy cấp quyết định:
- query -> next tool,
- query/memory -> sub-goal,
- query/memory -> stop-worthy context quality (gián tiếp qua verifier feedback).

Planner không trực tiếp execute tool; nó tối ưu “quyết định hành động”.

## 2) Input/Output contracts cần khóa chặt

Planner step output chuẩn:
- `context`: đủ dữ liệu để worker tạo command chính xác.
- `sub_goal`: hành động cụ thể, measurable.
- `tool_name`: phải thuộc allow-list.

Nếu 1 field trượt, toàn loop dễ fail.

## 3) Unit tests nên có cho Planner
1. Schema validity test:
- output parse được vào schema.

2. Tool validity test:
- `tool_name in available_tools`.

3. Context completeness test:
- context chứa biến/file/result mà sub-goal tham chiếu.

4. Step budget discipline test:
- planner không tạo plan lan man khi còn ít step.

## 4) Phương pháp tuning Planner-only

Role routing khuyến nghị:
- planner_main = trainable
- planner_fixed/verifier/executor = fixed endpoint

Lợi ích:
- giảm non-stationarity,
- tăng khả năng giải thích causal effect của planner tuning.

## 5) Planner metrics phải report riêng
- Tool Selection Accuracy.
- Step Efficiency (steps to solve).
- Plan Coherence (đo consistency giữa context/sub_goal/tool).
- Invalid Action Rate.
- Final task success.

## 6) Thực hành
- Chạy `01_planner_playground.ipynb`.
- Sửa prompt templates của planner.
- Chạy lại cùng seed và so kết quả trên cùng batch query.
