# 06. Troubleshooting + Failure Analysis (Thực Chiến)

## 1) Failure taxonomy theo sub-agent

### Planner failures
- chọn sai tool,
- context thiếu,
- sub-goal không executable,
- loop dài vô ích.

### Worker failures
- command parse fail,
- tool timeout,
- argument mismatch.

### Verifier failures
- STOP quá sớm,
- CONTINUE quá lâu.

### Generator failures
- answer format lệch,
- tóm tắt không bám evidence.

## 2) Quy trình debug chuẩn
1. Xem trace step-by-step.
2. Gắn lỗi vào đúng role trước khi sửa.
3. Chạy role-specific regression test.
4. Chỉ khi pass role tests mới chạy end-to-end.

## 3) Checklist reproducibility
- fixed seeds,
- fixed eval split,
- fixed endpoint versions,
- lưu full config snapshot,
- lưu commit hash code.

## 4) Ablation matrix tối thiểu
- baseline (no tuning),
- planner-only tuning,
- planner-only + reward shaping,
- planner-only + verifier prompt variants (verifier vẫn fixed model).

## 5) Anti-pattern cần tránh
- Tune nhiều role cùng lúc ngay từ đầu.
- Chỉ nhìn final accuracy.
- Không đo cost/latency.
- Không lưu trajectory artifacts.
