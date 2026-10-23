# oryxos
- URL: https://github.com/oryx-labs/oryxos
- Stars: 194 (tại thời điểm đọc)
- Đánh giá ngày: 23/10/2026

## Là gì (2 dòng)
"Distributed AI Agent OS" tự host, viết bằng Java (Spring Boot): định nghĩa agent bằng markdown/1 câu, chạy kèm memory, 24 tool built-in, MCP, skills, cron, notify, audit đầy đủ. Điểm mạnh nhất: **"one binary, zero ceremony"** — một file chạy là có REST API + web console + scheduler + sandbox trong một process.

## Kiến trúc multi-device (chi tiết vừa đủ)
- **"Distributed" ở đây hiện tại chủ yếu là khái niệm**, không phải cluster thật. Roadmap README ghi rõ:
  - **Phase 1 — Single-node Runtime Kernel ✅ (shipped)**: mọi thứ chạy trên 1 node.
  - **Track A · Distributed foundation** — "pluggable storage (SQLite → MySQL/PostgreSQL), stateless instances, DB-based cluster scheduling (xxl-job style)" — nằm trong mục **"Next — parallel tracks, pick one and start"**, tức **chưa ship**.
- Mâu thuẫn cần ghi nhận: mục Features "Dynamic Agent Management" có nhắc *"cluster mode uses version polling and periodic reconciliation"* — ám chỉ có code nhận biết cluster mode, nhưng không có tài liệu nào đã đọc mô tả cách dựng cluster thật; trạng thái thực tế của distributed foundation **chưa xác minh được đầy đủ**.
- Những gì chạy được hôm nay trên 1 máy: nhiều agent cùng tồn tại trên 1 instance; Provider Registry động lưu trong SQLite, CRUD runtime, hỗ trợ đổi base URL → về nguyên tắc có thể trỏ sang endpoint OpenAI-compatible (ví dụ OpenRouter), nhưng docs chỉ liệt kê DeepSeek/Qwen/OpenAI/Ollama — **OpenRouter chưa được xác minh rõ**.
- Có sẵn channel adapter Telegram (`oryxos-channel-telegram`) — khớp nhu cầu điều khiển qua Telegram của Mạnh.
- Agent-to-agent (A2A) cũng nằm trên roadmap, chưa ship.

## Yêu cầu triển khai
- Java 21 + Maven 3.9+ để build (repo có `.sdkmanrc` pin toolchain; Maven tự cài Node local để bundle admin UI lần build đầu).
- Hoặc Docker: repo có `Dockerfile` + `docker-compose.yml`.
- Storage mặc định SQLite; Helm chart có trong `charts/oryxos`.

## Docs có đủ để dựng không?
- README đầy đủ cho single-node: build, cấu hình LLM key, `bin/oryx-server start`, CLI, Docker.
- **Không đủ** cho multi-node: không có hướng dẫn dựng cluster (vì track distributed chưa ship).

## Verdict — Mạnh deploy được không?
**Được có điều kiện.** Chạy được **single-node** trên 1 máy (phù hợp A1 Flex 12GB hơn là VPS 1GB — Java 21 + Spring Boot không phải nhẹ cân; VPS 1GB RAM coi như không khả thi, chưa có số đo RAM cụ thể trong docs đã đọc). Có Telegram channel, REST API, memory/audit — đủ làm "não" cho 1 node. **Nhưng**: (1) "distributed" multi-machine là roadmap chưa ship — không giải quyết bài toán dàn máy của Mạnh theo nghĩa phân tán; muốn nhiều máy thì phải tự chạy nhiều instance độc lập, không có orchestration; (2) OpenRouter free chưa được docs xác nhận rõ, cần test thực tế với base_url.

## Nguồn (URL cụ thể đã đọc)
- https://github.com/oryx-labs/oryxos (README: "OryxOS is an open-source Distributed AI Agent OS"; Roadmap Phase 1 ✅ / track A Distributed foundation ở mục Next; module oryxos-channel-telegram; Quick Start Java 21/Maven)
