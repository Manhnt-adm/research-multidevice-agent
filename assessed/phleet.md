# phleet
- URL: https://github.com/anurmatov/phleet
- Stars: 21 (repo cá nhân, mới tạo 2026-04-05, license MIT)
- Đánh giá ngày: 23/10/2026

## Là gì (2 dòng)
Nền tảng multi-agent AI tự host, viết bằng .NET 10: mỗi agent là một Docker container chạy persistent AI process (Claude CLI, Codex app-server, hoặc Gemini CLI), phối hợp qua RabbitMQ, điều phối workflow bằng Temporal, memory dùng chung qua Qdrant, điều khiển qua Telegram + dashboard React.

## Kiến trúc multi-device
- **Không phải multi-device thật.** README ghi rõ: "Agents run as Docker containers on your host" — toàn bộ stack (orchestrator, agent containers, RabbitMQ, Temporal) chạy trên **một host Docker duy nhất**, nối nhau qua docker network `fleet-net`.
- Task flow: user DM co-CTO trên Telegram → co-CTO gọi `fleet-temporal-bridge` start Temporal workflow → workflow publish directive vào RabbitMQ queue của agent → worker agent xử lý → publish kết quả về qua RabbitMQ → workflow resume → báo lại qua Telegram.
- Khả năng công cụ tách lớp MCP: mọi năng lực (memory, telegram, temporal, browser) là MCP server riêng (`fleet-memory`, `fleet-playwright`, `fleet-telegram`, `fleet-temporal`), agent nào được phép gọi tool nào điều khiển bằng DB — ý tưởng hay nhưng MCP endpoints vẫn nằm trên cùng host, **không có doc nào hướng dẫn chạy đa máy**.
- Có `docs/comms-deployment.md` — "Fleet.Comms" là API HTTPS cho first-party client enroll device, nhưng chỉ mới làm đến **auth + session** ("An enrolled device holds a valid token and has nothing to talk to yet"), chưa phải device điều khiển đa máy. Đây là hướng đang phát triển, chưa dùng được.
- Điểm học được: UWE (Universal Workflow Engine) — workflow engine động kiểu `[Workflow(Dynamic = true)]` của Temporal, load cây step JSON từ DB, thêm workflow mới chỉ cần INSERT DB không cần rebuild; pattern Temporal durable workflow (retry delegate 5 phút, resume qua activity history) đáng tham khảo.

## Yêu cầu triển khai
- **~8 GB RAM và ~20 GB disk** cho full stack (theo README: MySQL, Qdrant, Temporal Postgres, MinIO, agent containers).
- Docker + Docker Compose 24+. Setup qua `./setup.sh` (wizard hỏi token).
- AI provider: **Claude (OAuth subscription), Codex (OAuth), Gemini (OAuth)** — cần credential dạng subscription cá nhân, không dùng API key free. Hoặc chạy local model qua Ollama/LM Studio (docs `providers/local-models.md` hỗ trợ `qwen3.8:27b`, `gpt-oss:20b`) — nhưng base prompt ~30k token, doc khuyên context ≥131072, tức cần máy GPU khỏe chạy được model 20-27B (tác giả dùng Mac Studio).
- Cần: 2 Telegram bot token (CTO bot + notifier bot), một GitHub App với private key.

## Docs có đủ để dựng không?
Đủ cho một host khỏe: `setup.sh` + `.env.example` + `seed.example.json` + `docker-compose.example.yml`, troubleshooting cho 4 lỗi thường gặp (OAuth hết hạn, Telegram 409, Temporal empty list), docs/providers chi tiết cho từng provider (cloud + local), docs/upgrading. Không clone/cài/chạy — chỉ đọc.

## Verdict — Mạnh deploy được không?
**KHÔNG.** Nói thẳng: Temporal + RabbitMQ + MySQL + Qdrant + Postgres + MinIO + whisper, bản thân requirement đã ghi ~8GB RAM — VPS 1GB của Mạnh không chứa nổi ngay cả khi tắt hết agent. Provider AI đều cần subscription trả phí (Claude Max / ChatGPT Plus / tài khoản Google cá nhân) — phương án local model thì cần máy GPU chạy Qwen3 27B/gpt-oss 20B, mà Mạnh không có. Kiến trúc single-host, không giải được bài toán điều phối đa máy (VPS + S8+ + laptop) của Mạnh. **Giá trị duy nhất: nguồn ý tưởng** — pattern durable workflow của Temporal, MCP capability matrix điều khiển bằng DB, và UWE (workflow engine động bằng JSON, không cần rebuild) đáng học cho conductor mới.

## Nguồn
- README (yêu cầu 8GB RAM, stack services, single-host, task flow): https://github.com/anurmatov/phleet
- Bài deep dive kiến trúc (RabbitMQ exchanges, agent lifecycle, MCP, Temporal, UWE, memory): https://dev.to/ggsa/phleet-architecture-deep-dive-5b8b
- docs/providers/local-models.md (chạy model local, base prompt ~30k token, cần ≥131072 context): https://github.com/anurmatov/phleet/tree/main/docs/providers
- docs/ (comms-deployment.md, conversation-protocol.md, first-party-api.md): https://github.com/anurmatov/phleet/tree/main/docs
- Repo server phụ của tác giả (Mac Studio làm inference server — cho thấy họ chạy model local trên máy Apple Silicon): https://github.com/anurmatov/mac-studio-server
