# Extension Guide

## A) Thêm tool mới
1. Tạo thư mục `agentflow/agentflow/tools/<tool_name>/`
2. Implement class `...Tool` trong `tool.py` với metadata đầy đủ
3. Bảo đảm có `TOOL_NAME` rõ ràng để Initializer map chính xác
4. Test bằng script tool test trước khi gắn vào training

## B) Thêm dataset mới
1. Chuẩn hóa về định dạng parquet mà training loader đọc được
2. Bổ sung trường `question`, `result`, `extra_info`
3. Test data sanity trước (null, max length, duplicate)

## C) Điều chỉnh reward
- Hiện tại reward thiên về correct/incorrect
- Hướng mở rộng:
  - reward theo efficiency (ít bước hơn)
  - reward theo tool reliability
  - reward shaping theo intermediate goals

## D) Tune cấu hình train
- File chính: `train/config.yaml`
- Nhóm tham số cần ưu tiên:
  - throughput: `data.train_batch_size`, `rollout.n`
  - stability: `lr`, `kl_loss_coef`, clip ratio
  - latency/memory: `max_prompt_length`, `max_response_length`, `gpu_memory_utilization`

## E) Checklist production-minded
- Timeout policy cho từng tool
- Retry/backoff cho API errors
- Guardrail cho generated command
- Observability: trace per step + structured metrics
