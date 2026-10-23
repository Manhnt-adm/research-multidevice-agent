# RobotFleet (therohangupta/robot-fleet)

- URL: https://github.com/therohangupta/robot-fleet
- Stars: 33 (tại thời điểm đọc)
- Đánh giá ngày: 23/10/2026

## Là gì (2 dòng)
Framework open-source lập kế hoạch & scheduling TẬP TRUNG cho fleet robot dùng LLM: Planner biến mục tiêu ngôn ngữ tự nhiên thành DAG task, Allocator gán task cho từng robot, Schedule Manager điều phối thực thi. Repo nhỏ (33 commits), còn ở mức demo nghiên cứu.

## Pattern điều phối đáng học cho fleet agent của Mạnh
Có 2 pattern tái dùng trực tiếp: (1) tách 3 vai trò swappable — Planner (goal → DAG task có dependency), Allocator (gán task, hỗ trợ cả LLM lẫn MILP tối ưu), Executor (chạy task, báo success/failure về để replan); (2) mô hình "world state" khai báo cập nhật real-time + cơ chế replanning khi executor báo lỗi. Ánh xạ sang fleet agent của Mạnh: conductor sinh DAG job → allocator chọn agent (VPS/S8+) → worker báo kết quả → conductor replan. Lưu ý: LLM planning mỗi lần tốn token, phù hợp làm não tập trung của conductor chứ không nên gọi trên từng worker.

## Verdict
**Có pattern dùng được** — kiến trúc Planner/Allocator/Executor + DAG có dependency + vòng lặp replan từ feedback là blueprint gọn cho conductor của Mạnh, dù repo gốc là cho robot vật lý.

## Nguồn (URL cụ thể đã đọc)
- https://github.com/therohangupta/robot-fleet (README đầy đủ: Architecture, Task Planner/Allocator/Status Manager, CLI robotctl)
