# symphony
- URL: https://github.com/GradientHQ/symphony
- Stars: 48 (tại thời điểm đọc)
- Đánh giá ngày: 23/10/2026

## Là gì (2 dòng)
Framework multi-agent phi tập trung (Python) cho edge devices không đồng nhất: task được decompose rồi định tuyến tới agent phù hợp qua "beacon broadcast" + capability matching, kết quả tổng hợp bằng Chain-of-Thought voting. Mục tiêu: chạy LLM nhẹ trên hardware tiêu dùng (RTX, Jetson, Apple M) thay vì server GPU đắt.

## Kiến trúc multi-device (ngắn gọn)
- Pipeline 3 giai đoạn: Planning (task decomposition) → Execution (beacon broadcast → capability matching → agent execution) → Voting (CoT voting ra kết quả cuối). (https://github.com/GradientHQ/symphony — README "Architecture Overview")
- Mỗi agent đăng ký với `capabilities` (vd: text-generation, reasoning, math), chạy model local (vd: Mistral-7B-Instruct, CUDA), lắng nghe trên port riêng (8001...); giao tiếp qua global broadcast hoặc neighbor P2P; User node submit task vào mạng. (https://github.com/GradientHQ/symphony — README "Quick Start", "Configuration Example", "Network Topologies")
- Không có coordinator trung tâm (trustless coordination); có decentralized ledger ghi capabilities/reputation.

## Yêu cầu triển khai (ngắn gọn)
- Python venv + `pip install -r requirements.txt`; mỗi node cấu hình YAML (model path, device cuda, port, neighbors). (https://github.com/GradientHQ/symphony — README "Installation")
- Yêu cầu phần cứng thực: **GPU tiêu dùng mạnh** (RTX 3060/4090, Jetson, Apple M) để chạy model 7B local — điện thoại S8+/S20FE và VPS 1GB RAM **không chạy nổi**; A1 Flex 12GB CPU cũng không có CUDA.
- **Mức độ chín muồi: repo còn sơ khai** — chỉ **3 commit** tổng cộng kể từ khi tạo (2025-08-26), 0 release, 0 tag. README quảng cáo video demo nhưng repo gần như chỉ có một lần push duy nhất. (https://github.com/GradientHQ/symphony — "Repository details": Commits: 3)

## Verdict — Mạnh deploy được không?
**Loại** — Hai lý do độc lập, cái nào cũng đủ: (1) yêu cầu GPU mạnh (CUDA/RTX/Jetson) để chạy model local, toàn bộ hạ tầng của Mạnh (VPS 1GB, điện thoại Termux, A1 Flex CPU) đều không đáp ứng; (2) repo còn sơ khai (3 commit, chưa release) — rủi ro cao, không có bằng chứng deploy thực tế, không phù hợp với nguyên tắc "không tin demo" của Mạnh. Ý tưởng beacon-routing/CoT-voting thì hay nhưng chỉ nên lấy làm **chất liệu nghiên cứu ý tưởng**, không phải để chạy.

## Nguồn (URL cụ thể đã đọc)
- https://github.com/GradientHQ/symphony (README đầy đủ — overview, kiến trúc, quick start, config; 48 stars, 3 commit, 0 release, license MIT)
