# Mastery Roadmap (2-4 tuần)

## Tuần 1: Understand & Reproduce
- Đọc `00`, `01`, `04`
- Chạy `quick_start.py` và xác nhận tool/engine hoạt động
- Trace 1 query từ planner đến verifier bằng log

## Tuần 2: Train Small
- Giảm cấu hình train để chạy được 1 cycle ngắn
- Phân tích rollout JSON:
  - tool nào được chọn nhiều nhất
  - tỉ lệ STOP sớm
  - lỗi command execution
- Tối ưu prompt planner/executor để giảm bước thừa

## Tuần 3: Reward & Stability
- Audit reward pipeline (judge consistency)
- Thử thay judge prompt hoặc model judge
- Theo dõi variance reward theo seed/batch

## Tuần 4: Extension
- Thêm 1 tool domain-specific
- Thêm 1 benchmark/task set mới
- Viết báo cáo so sánh trước/sau (success rate, cost, latency)

## Deliverables nên có
- 1 dashboard log + metrics
- 1 notebook/markdown phân tích lỗi chính
- 1 patch cải thiện rõ ràng (prompt/tool/reward/config)
