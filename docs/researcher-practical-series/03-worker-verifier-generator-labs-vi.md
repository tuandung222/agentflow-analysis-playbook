# 03. Worker/Verifier/Generator Labs

## 1) Lôi Worker (Executor) ra vọc

Mục tiêu:
- test command generation contract,
- test tool execution robustness,
- đo timeout/error profiles.

Checklist:
- command có `tool.execute(...)` chưa?
- args đúng type chưa?
- output serialize được không?
- execution có timeout guard không?

## 2) Lôi Verifier ra vọc

Mục tiêu:
- test stop policy quality độc lập với planner tuning.

Hai lỗi kinh điển:
- Early STOP: kết luận sớm, thiếu bằng chứng.
- Late CONTINUE: gọi tool thừa, tốn cost.

Metrics:
- stop precision,
- unnecessary continuation rate.

## 3) Lôi Generator ra vọc

Mục tiêu:
- kiểm tra chất lượng tổng hợp final/direct output.

Nếu generator fixed endpoint:
- output style ổn định hơn,
- dễ đo tác động planner (giảm nhiễu evaluation).

## 4) Vì sao cần tách lab riêng?

Vì planner failure và verifier/worker failure dễ bị lẫn. Nếu không tách lab:
- bạn sẽ tune planner để chữa lỗi thực ra do worker/verifier.

## 5) Thực hành
- Chạy notebook `02_subagents_playground.ipynb`.
- Tạo ít nhất 20 test cases có expected behavior cho mỗi role.
- Ghi confusion log role-wise thay vì chỉ overall success.
