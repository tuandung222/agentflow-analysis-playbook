# 03. Dataset và Benchmark cho bài toán LLM Planning

Mục tiêu của bạn là tune Planner, còn Worker/Verifier/Generator cố định. Vì vậy, dataset và benchmark cần nhấn mạnh:
- chất lượng quyết định hành động,
- khả năng chọn tool đúng lúc,
- kiểm soát độ dài và hiệu quả trajectory.

## 1) Tư duy chọn dataset cho Planner tuning

Planner không trực tiếp “trả lời cuối” mà quyết định **next action policy**.
Do đó dữ liệu cần tạo pressure lên:
1. Multi-step decomposition.
2. Tool selection under uncertainty.
3. Evidence aggregation before stopping.

## 2) Nhóm dataset nên dùng

## Nhóm A: In-repo baseline (dễ bắt đầu)
- `Natural Questions` (search-heavy, evidence retrieval)
- `DeepMath-103K` (structured reasoning)
- `AIME24` (validation khó, ngắn)

Ưu điểm:
- tương thích ngay với pipeline hiện tại.

Hạn chế:
- chưa phủ hết “planning with tool orchestration” kiểu dài hạn trong môi trường tương tác.

## Nhóm B: Multi-hop QA / retrieval planning
- Ví dụ hướng dữ liệu: Hotpot-style, 2-hop/3-hop QA, open-domain evidence chaining.

Mục đích:
- ép Planner học kế hoạch truy xuất thông tin theo nhiều bước.

## Nhóm C: Tool-use & programmatic reasoning
- Bài toán cần gọi code tool/calculator/search theo chuỗi.

Mục đích:
- đo chất lượng chọn tool + điều phối tool sequence.

## Nhóm D: Interactive/agent planning benchmarks
- Môi trường web/navigation/shopping/simulator (nếu bạn có infra phù hợp).

Mục đích:
- đánh giá planning policy trong long-horizon decision setting.

## 3) Benchmark dimensions cần đo

Đừng chỉ dùng final accuracy. Với planning research, cần tối thiểu các nhóm metric sau:

1. Outcome metrics
- `Final Answer Accuracy` / `Pass@1`

2. Planning metrics
- `Plan Success Rate`: tỉ lệ trajectory đạt mục tiêu với tools hợp lệ
- `Tool Selection Accuracy`: chọn đúng tool tại mỗi decision point (nếu có oracle/action labels)
- `Step Efficiency`: số bước trung bình đến khi solve

3. Control metrics
- `Verifier Stop Quality`: quality của stop timing (early-stop / late-stop)
- `Execution Validity`: tỉ lệ command chạy thành công

4. Cost/latency metrics
- token cost,
- API/tool cost,
- wall-clock latency.

## 4) Benchmark suite đề xuất cho luận án/bài báo

Một suite cân bằng có thể gồm:
- 40% search + multi-hop QA,
- 30% math/programmatic reasoning,
- 30% interactive tool-use tasks.

Rồi report theo cả 3 mức:
1. Overall score,
2. Domain-specific score,
3. Efficiency-adjusted score (accuracy per cost/time).

## 5) Dữ liệu cho tuning planner: schema gợi ý

Schema sample tối thiểu (dataset source):

```json
{
  "question": "...",
  "result": "groundtruth",
  "extra_info": {
    "index": 123,
    "domain": "search|math|tooluse",
    "difficulty": "easy|medium|hard"
  }
}
```

Dữ liệu train thực tế (sau rollout) trong pipeline `verl` sẽ là transition-level:
- `prompt_ids`,
- `response_ids`,
- reward cuối,
- metadata: `data_id`, `rollout_id`, `turn_index`.

## 6) Nếu bạn chỉ tune Planner và fix các role khác

Dataset nên ưu tiên task mà Planner có ảnh hưởng lớn:
- bài toán cần lựa chọn tool nhiều lần,
- bài toán có nhiều đường đi (có thể sai nếu chọn tool sai),
- bài toán mà stop decision ảnh hưởng outcome.

Không nên bắt đầu bằng task chỉ cần một câu trả lời trực tiếp (Planner leverage thấp).

## 7) Anti-pattern khi chọn benchmark

1. Chỉ đo final answer accuracy.
2. Không log tool-level failures.
3. Trộn domain quá dị thể nhưng không stratify report.
4. So sánh model mà không chuẩn hóa inference budget/time limit.
5. Thay đổi đồng thời Planner + Verifier + Executor trong cùng một experiment batch.

## 8) Protocol chọn benchmark thực dụng (khuyến nghị)

1. Pha 1 (stability): dùng in-repo data để pipeline ổn định.
2. Pha 2 (planning stress): thêm multi-hop + tool-sequence tasks.
3. Pha 3 (generalization): test chéo domain, giữ nguyên budget.
4. Pha 4 (paper-ready): report đầy đủ quality + efficiency + reliability.
