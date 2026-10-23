# skillhub (iflytek)

- URL: https://github.com/iflytek/skillhub
- Stars: 5,268 (tại thời điểm đọc)
- Đánh giá ngày: 23/10/2026

## Là gì (2 dòng)
Registry tự host (open source) để publish, version và quản trị tập trung các **skill package** của agent — ví như "kho skill riêng" của tổ chức, có RBAC, audit log và CLI cài skill theo từng agent platform. License Apache 2.0, deploy bằng Docker Compose hoặc Kubernetes.

## Kiến trúc multi-device / vai trò trong fleet (chi tiết vừa đủ)
- **Vai trò: kho skill dùng chung, không phải orchestrator.** Mỗi agent (kể cả chạy trên các thiết bị khác nhau) có thể trỏ CLI về cùng một registry trung tâm rồi `skillhub search` / `skillhub install <skill> --agent <platform>` để kéo skill về máy mình. README liệt kê tương thích với nhiều agent platform: OpenClaw, Hermes Agent, DeepSeek Harness, HarnessClaw Engine, AstronClaw, Loomy, astron-agent — và có lớp tương thích với client kiểu ClawHub. (https://github.com/iflytek/skillhub, mục README "Usage with Agent Platforms")
- Skill có semantic versioning, tag (`beta`/`stable`), auto-tracking `latest`, chia theo namespace (team/global) với role Owner/Admin/Member, review gate khi promote lên global scope; toàn bộ action quản trị đều có audit log. Điều này khớp với bài toán Mạnh đặt ra ở MVP: "agent không được tự chấm điểm", cần quản trị tập trung. (https://github.com/iflytek/skillhub, mục README "Highlights" và "Governance and Safety")
- API REST native + API token có scope; storage cắm được (filesystem local cho dev, S3/MinIO cho production). Backend Java 21 + frontend React, ảnh Docker cho `linux/amd64`, `linux/arm64`, `linux/riscv64` — bản arm64 tốt cho A1 Flex ARM của Mạnh. (https://github.com/iflytek/skillhub, mục "Container Runtime")
- Có thư mục `builtin-skills/`, `plugins/dsh`, `scanner/` (quét bảo mật package) và `charts/skillhub` (Helm) trong repo. (https://github.com/iflytek/skillhub, mục "Root files")
- Không tìm thấy bằng chứng skillhub điều phối agent giữa các thiết bị (không có scheduler/dispatcher như hatchet hay conductor) — nó chỉ cung cấp skill cho agent tự kéo về.

## Yêu cầu triển khai
- Điều kiện tiên quyết ghi trong README: Docker & Docker Compose. Lệnh quick start: `curl -fsSL https://imageless.oss-cn-beijing.aliyuncs.com/runtime.sh | sh -s -- up` (pull ảnh mới nhất từ GHCR). (https://github.com/iflytek/skillhub, mục "Quick Start" và "Prerequisites")
- Mức RAM/CPU tối thiểu **chưa xác minh được** trong phần đã đọc — không thấy con số yêu cầu tài nguyên cụ thể trong README.
- Lưu ý: script cài nhanh tải từ host Alibaba OSS tại Trung Quốc (oss-cn-beijing) — tuy là infra chính thức của repo nhưng nên cân nhắc, có thể thay bằng cách chạy `docker-compose.yml` trong repo trực tiếp để tránh `curl | sh`.

## Docs có đủ để dựng không?
Có vẻ đủ: README liệt kê User Guide (iflytek.github.io), Developer Docs (zread.ai), Python examples (`examples/python`), hướng dẫn CLI chi tiết, Kubernetes/Helm, smoke test, monitoring. (https://github.com/iflytek/skillhub, mục "Documentation")

## Verdict — có dùng được cho Mạnh không? (Được / Được có điều kiện / Loại + 1–3 dòng lý do)
**Được có điều kiện.** Apache 2.0, mở, tương thích chuẩn ClawHub-style, skill là gói `SKILL.md` chuẩn — không phải hệ sinh thái đóng của iFlytek. Nhưng với fleet hiện tại (vài máy, ít skill) thì chạy cả registry server là thừa; chỉ nên dựng khi số skill/agent tăng lên, và đặt trên A1 Flex 12GB sau Tailscale — không nên đặt trên VPS Oracle 1GB. Khả năng ZeroClaw kéo skill trực tiếp từ skillhub chưa được xác minh.

## Nguồn (URL cụ thể đã đọc)
- https://github.com/iflytek/skillhub (README + repo tree, stars/license/topics)
- http://dev.to/fenju_fu/when-agents-become-a-fleet-you-need-a-skill-registry-not-just-memory-3l93 (vai trò skill registry trong fleet)
