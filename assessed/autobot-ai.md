# AutoBot-AI
- URL: https://github.com/mrveiss/AutoBot-AI
- Stars: 17 (tại thời điểm đọc; 16.448 commits — repo solo-author nhưng số commit bất thường lớn)
- Đánh giá ngày: 23/10/2026
## Là gì (2 dòng)
Nền tảng AI tự host dạng monorepo lớn (Apache-2.0) gồm 3 lớp: Platform Core (chat, RAG + knowledge graph, LLM gateway), Service Lifecycle Manager (SLM — deploy/vận hành/scale stack), và Modules cài thêm (vd "AutoBot LLC" — agent-company tự trị). Kèm cả voice, vision-driven browser/computer control, RBAC, governance.
## Kiến trúc multi-device (chi tiết)
- Mô hình **SLM control plane + fleet nodes**: SLM (Postgres + backend + nginx, cài bằng one-line installer) "provisions the platform onto your fleet nodes" — **deploy stack (vector DB, cache, database, inference, workers) qua Ansible** ("Stand up the full stack from a blank host or across a fleet (Ansible)"); wizard hỗ trợ "adding fleet nodes, enrolling agents, assigning roles".
- "Multi-server" ở đây = **SLM quản lý vòng đời dịch vụ của chính nó** trên nhiều máy (deploy/upgrade/health monitoring/scale, thêm node/NPU worker), chứ KHÔNG phải orchestrator phân phối task cho agent/worker tự do như hatchet hay conductor của Mạnh. Khả năng tái dùng Ansible/SLM để quản lý fleet worker generic (ngoài stack AutoBot) — **chưa xác minh được** từ nguồn đã đọc.
- Mục README "Example: Managing a Fleet" và tài liệu SLM (docs/) — **chưa đọc được** (trang bị giới hạn tốc độ khi đọc tiếp). Chi tiết rolling/canary/blue-green deployment — **chưa xác minh được**.
- Số liệu 17 stars với 16.448 commits của một tác giả cá nhân là dấu hiệu cần thận trọng (có thể commit bot-generated) — chất lượng thực tế của Ansible playbooks chưa được kiểm chứng.
## Yêu cầu triển khai
- **SLM node tối thiểu: 1.5 GB RAM** (bản native; installer kiểm tra) — VPS Oracle 1GB của Mạnh KHÔNG ĐẠT ngưỡng cài tối thiểu.
- **Platform node tối thiểu: 8 GB RAM (khuyến nghị 16+ GB)**, 4–8+ cores, Docker 24/25+, Ubuntu 22.04 — A1 Flex 12GB chỉ đạt mức tối thiểu, không đạt mức khuyến nghị, và còn phải gánh vector DB + inference + workers + Postgres cùng lúc.
- LLM: gateway provider-agnostic + fallback, hỗ trợ local inference (có worker GPU/NPU riêng) — nhưng stack local inference đi kèm là đúng phần tốn RAM nhất.
## Docs có đủ để dựng không?
README chính dài, chi tiết (Quick Start 2 option, bảng yêu cầu hệ thống rõ ràng, INSTALL.md, Capability Catalog...). INSTALL.md, docs/ và phần Example fleet chưa đọc được trong đợt này. Đánh giá: docs có vẻ đầy đủ về mặt hình thức, nhưng quy mô monorepo + 17 stars khiến việc "dựng lên chạy thật" rủi ro cao.
## Verdict — Mạnh deploy được không? (Được / Được có điều kiện / Loại + 1–3 dòng lý do)
**Loại.** (1) Quá nặng cho toàn bộ hạ tầng của Mạnh: SLM node cần ≥1.5GB RAM (VPS 1GB loại ngay), platform node cần 8GB tối thiểu/16GB khuyến nghị — A1 Flex 12GB chạy stack này (vector DB + inference + Postgres + workers) sẽ chật. (2) Ansible fleet management của nó phục vụ vòng đời stack AutoBot, không phải nền tảng điều phối agent/worker đa thiết bị theo nhu cầu của Mạnh — học hỏi ý tưởng thì được, deploy thì sai bài toán.
## Nguồn (URL cụ thể đã đọc)
- https://github.com/mrveiss/AutoBot-AI (README: mô tả 3 lớp Core/SLM/Modules, SLM control plane + Ansible deploy across fleet, bảng System Requirements: SLM node 1.5GB RAM min / platform node 8GB min–16GB rec, Apache-2.0, stars 17, 16.448 commits)
