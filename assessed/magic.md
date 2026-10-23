# MagiC
- URL: https://github.com/kienbui1995/magic
- Stars: 0 (fork 0, watcher 0 — đọc trực tiếp từ trang repo; mới tạo 16/03/2026, 155 commits, 10 releases)
- Đánh giá ngày: 23/10/2026
## Là gì (2 dòng)
MagiC là "Kubernetes for AI agents" theo lời tác giả: một server Go quản trị fleet AI agent — worker đăng ký qua protocol HTTP mở (MCP²), server định tuyến task theo capability, kiểm soát chi phí LLM, chạy workflow DAG, circuit breaker, approval gate. Khác CrewAI/LangGraph (chuyên build agent), MagiC chỉ lo tầng quản trị/điều phối agent đa ngôn ngữ/framework.
## Kiến trúc multi-device
- Server Go duy nhất (binary nhỏ, "zero external Go dependencies" theo bài dev.to) lắng nghe :8080; worker (Python/Go/TS SDK hoặc bất kỳ HTTP client nào) đăng ký bằng cách POST `worker.register` kèm tên, URL endpoint của chính nó, và danh sách capabilities (ví dụ `summarize`, `content_writing`).
- Worker phải gửi heartbeat mỗi 30s (`worker.heartbeat` kèm `current_load`); server health-check đánh dấu worker offline nếu mất heartbeat (nhưng không đánh offline khi worker còn task đang chạy).
- Task gửi qua `POST /api/v1/tasks` với `required_capabilities`; Router chọn worker theo strategy (`best_match`, `cheapest`...), Dispatcher gửi HTTP POST `task.assign` tới endpoint của worker, worker trả `task.complete`/`task.fail`.
- Workflow DAG: steps có `depends_on`, chạy song song các nhánh độc lập, failure handling từng step (`retry`/`skip`/`abort`/`reassign`), có approval gate chờ human trước khi tiếp tục.
- Cost Controller: theo dõi spend per worker/team/model, alert ở 80% budget, auto-pause worker ở 100%; worker tự báo `cost` trong `task.complete`.
- Circuit breaker: worker fail 3 lần liên tiếp → server ngừng gửi task cho nó 30 giây (theo bài dev.to).
- LLM Gateway (từ 0.8.0): routing multi-provider OpenAI/Anthropic/**Ollama** với strategy `cheapest`/`fastest`/`best`, fallback tự động, đếm token + tính chi phí — Ollama có nghĩa chạy model local không tốn tiền API.
- Storage: in-memory (dev), SQLite file (`MAGIC_STORE`), PostgreSQL optional — SQLite đủ cho single instance.
- Có dashboard web `/dashboard`, Prometheus `/metrics`, SSE streaming, RBAC, policy engine, audit log, Docker image multi-arch amd64/arm64.
## Yêu cầu triển khai
- Server: build Go 1.25+ (`go build ./cmd/magic`) hoặc `docker run -p 8080:8080 ghcr.io/kienbui1995/magic:latest` với `MAGIC_API_KEY` (tối thiểu 32 ký tự) + `MAGIC_STORE=/data/magic.db` cho persistence. Không cần Postgres.
- Worker: Python 3.11+ (`pip install magic-ai-sdk`) hoặc Go/TS SDK; worker phải mở một HTTP endpoint mà server với tới được (server PUSH task tới worker, không phải worker pull). Worker gửi heartbeat 30s/lần.
- Bảo mật sản xuất: chỉ có API key auth; docs deployment khuyên đặt sau nginx/Cloudflare và thêm rate limit ngoài (built-in token bucket chỉ per-endpoint).
## Docs có đủ để dựng không?
Đủ để dựng thử: README có quick start đầy đủ (pip install / build từ source / Docker / one-click Railway-Render-Fly), docs-site VitePress có guide deployment, protocol (format message MCP², vòng đời register→heartbeat→assign→complete/fail), storage, observability, policies, RBAC; CHANGELOG chi tiết từng version 0.1.0→0.8.0 (2026-03-16→2026-04-14). Thiếu: hướng dẫn TLS/reverse-proxy cụ thể, và không có tài liệu triển khai worker trên thiết bị di động/NAT — phải tự suy ra (worker cần endpoint reachable).
## Verdict — Mạnh deploy được không?
Deploy được về mặt kỹ thuật trên hạ tầng hiện tại: Go single binary + SQLite chạy ngon trên VPS 1GB; worker Python chạy trên điện thoại qua Termux, khai báo endpoint qua IP Tailscale để server với tới; LLM gateway hỗ trợ Ollama nên không cần subscription AI trả phí (chạy model free qua OpenRouter cũng là worker HTTP bình thường). NHƯNG rủi ro lớn: repo mới 1 tháng tuổi, 0 stars, 1 tác giả — chưa có cộng đồng, chưa được ai kiểm chứng production, roadmap còn dài. Dùng để học/hack/prototype thì tốt; đừng đặt toàn bộ conductor thật lên nó cho tới khi Mạnh tự test kỹ.
## Nguồn
- https://github.com/kienbui1995/magic (README — kiến trúc, API, DAG, config; trang repo — stars/forks/commit count)
- https://github.com/kienbui1995/magic/blob/main/CHANGELOG.md (tính năng 0.1.0→0.8.0, Docker multi-arch amd64/arm64, health check, SQLite/Postgres)
- https://github.com/kienbui1995/magic/blob/main/docs-site/guide/deployment.md (env vars, storage backends, Docker, compose với pgvector)
- https://github.com/kienbui1995/magic/blob/main/docs-site/guide/protocol.md (MCP² message format, worker.register, heartbeat 30s, task.assign/complete/fail)
- https://dev.to/kienbui1995/why-i-built-magic-and-why-managing-ai-agents-is-the-real-problem-37i9 (bài tác giả — lý do build, circuit breaker 3 fails/30s, cost alert 80%/auto-pause 100%, Go single binary, Apache 2.0)
- https://github.com/kienbui1995/magic/tree/main/docs-site/guide (danh mục guide: python/go SDK, rbac, policies, streaming, webhooks)
