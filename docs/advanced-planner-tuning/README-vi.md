# Tutorial Chuyên Sâu: Tuning Planner cho Hệ Multi-Agent Tool-Use

Bộ tutorial này dành cho PhD/AI Researchers đang nghiên cứu **LLM planning** và muốn tối ưu theo hướng:
- Chỉ tune `Planner` bằng RL (`verl`),
- Giữ `Worker/Executor`, `Verifier`, `Generator` cố định qua inference endpoint có sẵn,
- Đo lường planning quality rõ ràng ở cả cấp final answer và cấp trajectory.

## Bối cảnh và giả định chính
- Framework gốc: AgentFlow (`Planner -> Executor -> Verifier -> Generator` logic).
- Hệ huấn luyện: `verl` + Ray + vLLM/endpoint.
- Cấu hình nghiên cứu mục tiêu:
  - `Planner main`: trainable
  - `Worker(Executor)`: fixed
  - `Verifier`: fixed
  - `Generator`: fixed

## Nội dung tutorial
1. `01-system-contracts-vi.md`
   - Thiết kế từng sub-agent, contract input/output theo method level.
2. `02-orchestration-loop-vi.md`
   - Orchestration loop chi tiết, pseudocode, state machine và sequence diagram.
3. `03-datasets-benchmarks-vi.md`
   - Chọn dataset cho bài toán planning và benchmark/metrics để đo đúng bản chất planning.
4. `04-planner-tuning-protocol-vi.md`
   - Protocol tuning Planner khi các sub-agent khác bị cố định.
5. `05-experiment-playbook-vi.md`
   - Experimental design, ablation, failure analysis, và checklist reproducibility.

## Kết quả kỳ vọng sau khi học xong
- Xây được pipeline planner-tuning có thể publish-level:
  - objective rõ,
  - benchmark hợp lý,
  - ablation có sức thuyết phục,
  - phân tích lỗi trajectory ở mức khoa học.
