# AgentFlow Analysis Playbook

Repository này dành cho người mới muốn:
- Hiểu mã nguồn gốc `AgentFlow` (Stanford/Lupantech, ICLR 2026)
- Nắm nền tảng RL training cho LLM theo hướng **agentic planning + tool orchestration**
- Có lộ trình thực chiến để đọc, chạy, debug, và mở rộng hệ thống

## Mục tiêu của repo
- Tách bạch phần **khái niệm** và **phân tích mã nguồn theo luồng chạy**
- Giải thích rõ luồng dữ liệu train/infer trong AgentFlow
- Chỉ ra các điểm dễ lỗi với người mới (GPU, Ray, vLLM, tool-calling)

## Đối tượng
- Kỹ sư mới bắt đầu với RLHF/GRPO
- Người muốn xây hệ thống LLM có Planner-Executor-Verifier
- Team cần tài liệu onboarding nội bộ

## Cấu trúc tài liệu
- `docs/00-source-summary.md`: Tổng quan nhanh về repo gốc
- `docs/01-system-architecture.md`: Kiến trúc module và vòng đời suy luận
- `docs/02-flow-grpo-training.md`: Luồng train Flow-GRPO/verl
- `docs/03-hands-on-setup.md`: Cách cài đặt, chạy thử và kiểm tra môi trường
- `docs/04-concepts-primer.md`: Kiến thức nền RL cho LLM và orchestration
- `docs/05-mastery-roadmap.md`: Lộ trình học 2-4 tuần để master codebase
- `docs/06-extension-guide.md`: Cách thêm tool, dataset, reward, và benchmark

## Repo gốc được phân tích
- Source: https://github.com/lupantech/AgentFlow
- Snapshot được clone cục bộ tại: `/Users/admin/TuanDung/repos/AgentFlow`

## Thứ tự đọc khuyến nghị
1. `docs/04-concepts-primer.md`
2. `docs/00-source-summary.md`
3. `docs/01-system-architecture.md`
4. `docs/03-hands-on-setup.md`
5. `docs/02-flow-grpo-training.md`
6. `docs/06-extension-guide.md`
7. `docs/05-mastery-roadmap.md`

## Phạm vi và giới hạn
- Phạm vi: phân tích thiết kế, luồng chạy, thực thi và điểm mở rộng kỹ thuật
- Giới hạn: không sao chép toàn bộ phần cài đặt của repo gốc
