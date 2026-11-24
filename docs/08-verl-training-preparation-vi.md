# Chuẩn bị file để train RL với `verl` (và cách AgentFlow đang adapt)

## 1) Cần chuẩn bị những nhóm file nào?

### A. File cấu hình huấn luyện
Mục đích: định nghĩa toàn bộ tham số RL, rollout, model, dữ liệu, hạ tầng.

- `train/config.yaml`
  - Biến môi trường: GPU, key API, model base, tool config.
  - Hyperparameters: batch size, rollout count, learning rate, KL, epoch.
  - Đường dẫn dữ liệu train/val parquet.
- `agentflow/verl/config.yaml`
  - Hydra config bridge sang `verl`.
  - Cấu hình rollout mode (`async`) và custom async server cho agent mode.

### B. File entrypoint và launcher
Mục đích: biến config thành lệnh chạy thực tế và bootstrap pipeline RL.

- `train/train_agent.py`
  - Đọc `train/config.yaml`, export env vars, build command `python -m agentflow.verl ...`.
- `agentflow/verl/entrypoint.py`
  - Entry chính của `verl` trong AgentFlow.
  - Init Ray, load tokenizer/processor, khởi tạo `AgentFlowTrainer`.

### C. File rollout agent (phần khác biệt lớn nhất khi dùng `verl` cho agentic workflow)
Mục đích: biến một sample thành trajectory có tool-calling, rồi trả reward cho RL.

- `train/rollout.py`
  - Định nghĩa `Rollout(LitAgent)` để thực thi query với solver.
  - Trích xuất answer, gọi reward, ghi artifact rollout JSON.
  - Dùng `FileLock` để đồng bộ step folder giữa worker async.
- `train/utils.py`
  - Hàm chấm điểm (`compute_score`) hiện dùng LLM judge.

### D. File trainer adapter và dataset adapter cho `verl`
Mục đích: nối “rollout ngoài môi trường/tool” với batch/token pipeline của PPO.

- `agentflow/verl/trainer.py`
  - `AgentFlowTrainer` kế thừa `RayPPOTrainer`.
  - Dùng `AgentModeDaemon` để gửi task sang server, thu rollout, dựng batch train/val.
- `agentflow/verl/dataset.py`
  - `AgentDataset` kế thừa `RLHFDataset`.
  - Patch dữ liệu để tương thích `DataProto` (`fake_ids`, index handling).

### E. File inference/system core (được rollout gọi trực tiếp)
Mục đích: thực thi trajectory thực tế Planner -> Executor -> Verifier.

- `agentflow/agentflow/solver.py`
- `agentflow/agentflow/models/planner.py`
- `agentflow/agentflow/models/executor.py`
- `agentflow/agentflow/models/verifier.py`
- `agentflow/agentflow/models/memory.py`

### F. File dữ liệu
Mục đích: cung cấp train/val samples theo định dạng đọc được bởi pipeline.

- `data/get_train_data.py` -> tạo train parquet.
- `data/aime24_data.py` -> tạo validation parquet.
- Dữ liệu đầu ra thường là:
  - `data/train/combined_train.parquet`
  - `data/val/aime24.parquet`

### G. File vận hành train/serve
Mục đích: chạy server rollout và train process độc lập, có log xoay vòng.

- `train/serve_with_logs.sh`
- `train/train_with_logs.sh`


## 2) Về mặt logic, để train RL trên `verl` tối thiểu cần gì?

Nếu tách theo vai trò kỹ thuật, tối thiểu bạn cần:

1. `Dataset` tương thích loader (`RLHFDataset`/DataProto path).
2. `Entrypoint` để tạo worker groups, reward manager, dataloaders, trainer.
3. `Trainer` (mặc định hoặc custom) để chạy loop PPO/GRPO.
4. `Rollout`/environment bridge để sinh trajectory từ policy.
5. `Reward` function để trả score cho mỗi sample/trajectory.
6. Cấu hình Hydra/YAML để map toàn bộ tham số vào `verl`.

AgentFlow có đủ cả 6 phần này, và phần custom chính nằm ở bridge agent mode (`rollout.py`, `agentflow/verl/trainer.py`).


## 3) Hiện tại AgentFlow đang adapt với `verl` như thế nào?

## 3.1 Pipeline thực thi

1. `train/train_agent.py` đọc `train/config.yaml`.
2. Script build command `python -m agentflow.verl key=value ...`.
3. `agentflow/verl/entrypoint.py`:
   - init Ray.
   - tạo tokenizer/processor.
   - tạo worker mapping Actor/Critic/(Ref/RewardModel).
   - load train/val dataset qua `AgentDataset`.
   - khởi tạo `AgentFlowTrainer` và gọi `fit()`.

## 3.2 Cách rollout được “agent hóa”

- `train/rollout.py` không chỉ generate text như PPO LLM thường.
- Nó gọi full solver (Planner/Executor/Verifier + tools), tức là rollout có nhiều bước tool-calling.
- Kết quả trajectory được lưu JSON để phục vụ debug và kiểm toán.

## 3.3 Cách reward đang làm

- Reward hiện thiên về correctness 0/1 (`train/utils.py`).
- Judge dựa trên LLM scoring thay vì rule-based thuần.
- Đây là cách dễ chạy, nhưng có thể gây reward noise và chi phí API.

## 3.4 Cách data được “vá” để khớp `verl`

- `agentflow/verl/dataset.py` thêm `fake_ids` vì DataProto cần ít nhất một tensor.
- Đồng thời đưa index vào record để tracking sample/rollout.

## 3.5 Cách trainer được custom cho agent mode

- `agentflow/verl/trainer.py` dùng `AgentModeDaemon` để:
  - queue task.
  - theo dõi rollout completion.
  - thu triplets và convert về batch PPO.
- Validation chấp nhận partial completion (có ngưỡng tối thiểu), phù hợp môi trường tool/API không ổn định tuyệt đối.


## 4) Checklist chuẩn bị khi bạn tự build repo RL + `verl` theo kiểu AgentFlow

- [ ] Có train/val parquet schema rõ ràng (`question`, `result`, metadata cần thiết).
- [ ] Có rollout class gọi được “môi trường thật” (tool/API/simulator).
- [ ] Có reward function rõ, deterministic càng tốt (hoặc có cơ chế giảm noise).
- [ ] Có trainer adapter để chuyển rollout output -> PPO batch.
- [ ] Có config tập trung (YAML/Hydra) để reproducible.
- [ ] Có scripts để chạy service và training tách riêng + logging.
- [ ] Có cơ chế timeout/retry/file-lock để chịu được async/multi-worker.


## 5) Nhận xét ngắn về cách làm hiện tại

Điểm tốt:
- Kiến trúc bridge với `verl` rõ ràng, tách được rollout environment khỏi PPO core.
- Dễ mở rộng tools và model-engine split (trainable vs fixed).

Điểm cần cải thiện nếu muốn production:
- Giảm phụ thuộc reward judge bằng LLM (chi phí + variance).
- Harden phần generated command execution (sandbox/DSL).
- Bổ sung đánh giá trajectory-level metrics (không chỉ final correctness).
