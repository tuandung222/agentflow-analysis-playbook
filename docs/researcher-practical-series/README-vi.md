# Series Thực Chiến Cho PhD/AI Researcher: Vọc Sâu AgentFlow

Series này tập trung vào thao tác thực chiến trên mã nguồn AgentFlow:
- Bóc tách từng sub-agent (`Planner`, `Worker/Executor`, `Verifier`, `Generator`).
- Rebuild orchestration loop trong Jupyter notebook.
- Thiết kế và chạy protocol finetune/tuning cho `Planner`.
- Đo benchmark planning bằng metrics trajectory-level, không chỉ final accuracy.

## Đối tượng
- PhD students làm RL for LLM, Tool-use, Planning.
- AI Researchers muốn mở paper direction về in-the-loop planning optimization.

## Cấu trúc series
1. `01-roadmap-vi.md`
2. `02-planner-deep-dive-vi.md`
3. `03-worker-verifier-generator-labs-vi.md`
4. `04-orchestration-notebook-implementation-vi.md`
5. `05-planner-finetune-notebook-vi.md`
6. `06-experiment-troubleshooting-vi.md`

## Notebook labs đi kèm
- `notebooks/researcher-practical-series/01_planner_playground.ipynb`
- `notebooks/researcher-practical-series/02_subagents_playground.ipynb`
- `notebooks/researcher-practical-series/03_orchestration_rebuild.ipynb`
- `notebooks/researcher-practical-series/04_planner_finetune_eval.ipynb`

## Cách dùng nhanh
1. Đọc `01-roadmap-vi.md` để định hướng.
2. Chạy notebook 01 -> 04 theo thứ tự.
3. Ghi lại experiment log bằng cùng format trong notebook 04.
4. Lặp ablation theo checklist ở `06-experiment-troubleshooting-vi.md`.
