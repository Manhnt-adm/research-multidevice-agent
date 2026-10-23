# Octop
- URL: https://github.com/TencentCloud/Octop
- Stars: 10761 (tại thời điểm đọc)
- Đánh giá ngày: 23/10/2026
## Là gì (2 dòng)
AI assistant tự host của TencentCloud (MIT, Python 3.12+ / FastAPI), chạy toàn bộ trong một process duy nhất: web dashboard, CLI, kênh IM (có Telegram), cron — chia sẻ một control-plane DB (SQLite mặc định, Postgres tùy chọn). Quảng bá là "multi-user, multi-agent" nhưng bản chất là trợ lý cá nhân/nhóm nhỏ.
## Kiến trúc multi-device (chi tiết)
- **"Multi-agent" ở đây là multi-agent TRONG một máy:** mỗi user có nhiều "experts" (agent chuyên gia) chạy trong cùng process; **AgentTeams (Beta)** = một coordinator lên lịch cho nhiều experts — tất cả vẫn là in-process, không có worker trên máy khác (nguồn: README mục Highlights/Features, https://github.com/TencentCloud/Octop).
- **"Cloud collaboration":** chỉ là nối hai instance Octop để "browse peer experts và chat across instances" (liên kết chat, history nằm ở phía expert) — không phải phân phối task/execution ra nhiều máy.
- **"Cloud–edge continuum"** (chạy Octop local, offload task nặng lên cloud) mới chỉ ở mục **Planned** trong Roadmap — CHƯA SHIPPED.
- **ACP runners (outbound):** Octop ủy thác task coding cho OpenCode/Claude Code/Codex qua Agent Client Protocol (stdio). Tài liệu setup chi tiết nằm ở docs/acp.md — chưa xác minh được từ các nguồn đã đọc liệu runner có thể nằm trên máy khác (không phải localhost) hay không.
- Không có khái niệm node/worker fleet, heartbeat, hay task routing đa máy như hatchet — kiến trúc "single, restart-safe process" (nguyên văn README).
- Remote desktop: điều khiển desktop của chính host qua dashboard — một máy.
## Yêu cầu triển khai
- macOS/Linux/Windows; installer one-line (curl từ Tencent COS) tự provision Python 3.12 venv bằng uv vào `~/.octop/`; có thư mục docker/ (README ghi "Docker (recommended for production)" — chi tiết chưa đọc).
- Prerequisites: "a modern multi-core CPU with a few GB of RAM for the process plus model/embedding caches" — vài GB RAM trở lên; LLM chạy qua provider (danh sách provider hỗ trợ nằm ở mục README "Supported LLM providers" — chưa xác minh được nội dung, trang bị 429 khi đọc tiếp).
## Docs có đủ để dựng không?
Một phần: README dài, có Quick Start, CLI reference, Architecture, Config; phần tiếp theo (providers, architecture detail, docs/expert-teams.md, docs/acp.md) không đọc được trong đợt này do bị giới hạn tốc độ truy cập. Với máy Linux 64-bit thông thường thì README phần đã đọc đủ để cài, nhưng câu hỏi multi-device cần docs/acp.md — chưa xác minh được.
## Verdict — Mạnh deploy được không? (Được / Được có điều kiện / Loại + 1–3 dòng lý do)
**Loại** (cho track multi-device). Multi-agent của Octop chạy trong một process một máy — không phân phối agent ra fleet của Mạnh; tính năng offload cloud↔edge thật sự (cloud–edge continuum) mới ở roadmap, chưa có. Nếu chỉ cần "một trợ lý tự host nặng ký trên A1 Flex" thì Octop deploy được (chạy sau Tailscale, LLM qua OpenRouter), nhưng nó nặng hơn ZeroClaw (vài GB RAM theo README vs ~100MB ZeroClaw đang chạy ổn) và trùng chức năng với bot hiện tại của Mạnh.
## Nguồn (URL cụ thể đã đọc)
- https://github.com/TencentCloud/Octop (README: mô tả, stars 10761, MIT, Python 3.12+, FastAPI+uvicorn, single process, SQLite/Postgres, AgentTeams Beta, cloud collaboration, roadmap planned gồm cloud–edge continuum, remote desktop, Telegram channel, prerequisites "few GB of RAM")
