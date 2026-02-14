# 01. Roadmap Thực Chiến (6-8 tuần)

## Tuần 1: System mapping
- Đọc source map + contract của Planner/Executor/Verifier.
- Chạy notebook `01_planner_playground.ipynb`.
- Mục tiêu: hiểu planner output schema và lỗi thường gặp.

## Tuần 2: Sub-agent dissection
- Chạy notebook `02_subagents_playground.ipynb`.
- Bóc tách worker/verifier/generator bằng mock + contract test.
- Mục tiêu: hiểu dependency giữa các role.

## Tuần 3: Orchestration re-implementation
- Chạy notebook `03_orchestration_rebuild.ipynb`.
- Tự hiện thực state machine loop (planning -> tool -> verify -> stop).
- Mục tiêu: tự cầm loop, không phụ thuộc black-box.

## Tuần 4-5: Planner tuning protocol
- Chạy notebook `04_planner_finetune_eval.ipynb`.
- Thiết kế experiment matrix cho planner-only tuning.
- Mục tiêu: setup lặp thí nghiệm reproducible.

## Tuần 6: Planning benchmarks
- Chọn benchmark theo domain:
  - retrieval planning,
  - tool-use planning,
  - math/program planning.
- Mục tiêu: report đủ quality + efficiency + reliability.

## Tuần 7-8: Paper-grade analysis
- Làm ablation theo reward, verifier strictness, tool subsets.
- Viết failure taxonomy ở cấp trajectory.
- Mục tiêu: có report publish-ready.

## Deliverables tối thiểu
- 1 bảng benchmark chính + CI.
- 1 dashboard trajectory diagnostics.
- 1 notebook tái lập orchestration loop.
- 1 protocol planner-only tuning có thể rerun end-to-end.
