# figaro
- URL: https://github.com/byt3bl33d3r/figaro
- Stars: 155
- Đánh giá ngày: 23/10/2026
## Là gì (2 dòng)
Figaro là hệ điều phối "fleet" agent Claude Computer Use trên môi trường desktop đầy đủ: các agent nhìn màn hình, click/gõ phím như người thật. Supervisor phân công task cho worker, toàn bộ giao tiếp qua NATS (pub/sub + JetStream), quản lý qua dashboard hoặc chat Telegram với supervisor.
## Kiến trúc multi-device
- **Bus trung tâm: NATS.** Orchestrator, worker, supervisor, gateway, UI đều nói chuyện qua NATS (Core NATS cho đăng ký/presence/task assignment point-to-point `figaro.worker.{id}.task`, JetStream stream TASKS cho event bền vững, request/reply cho API). Worker đăng ký qua `figaro.register.worker`, heartbeat giữ liveness; orchestrator claim worker idle rồi giao task.
- **Worker là agent computer-use chạy Claude agent SDK** (Bun binary), thấy desktop qua screenshot, di chuột/click/gõ phím, chạy shell, stream tiến trình qua JetStream. Trong Docker: desktop Linux containerized (Fluxbox + TigerVNC + Chromium + noVNC); nhưng worker cũng chạy độc lập trên mọi máy có môi trường desktop.
- **"Agent Upgrade Path" cho máy vật lý/VM:** pre-register desktop qua UI ("Add Desktop") hoặc biến môi trường `FIGARO_DESKTOP_WORKERS` với URL `vnc://`, `rdp://`, `ssh://`, `telnet://`, `ws(s)://` (macOS/Windows/Linux đều được). Ban đầu desktop chỉ ở chế độ view-only để supervisor quan sát/tương tác qua VNC; khi `figaro-worker` chạy trên máy đó với ID trùng khớp, entry tự nâng lên worker agent đầy đủ — ngắt kết nối thì revert về view-only, không cần cấu hình lại.
- **Supervisor** = cùng binary worker chạy với flag `--supervisor`, nhận task phức tạp rồi delegate cho worker, có thể trực tiếp screenshot/click trên bất kỳ desktop nào qua VNC tools, hoặc chạy lệnh SSH/telnet.
- **Desktop streaming:** Apache Guacamole (guacd) qua WebSocket, dashboard React xem trực tiếp grid desktop.
- **Gateway multi-channel:** hiện tại Telegram — Mạnh có thể chat tự nhiên để tạo task, schedule cron, hỏi trạng thái; gateway còn xử lý help-request (human-in-the-loop) và transcribe voice message (dùng Claude STT endpoint qua OAuth từ `~/.claude/.credentials.json`).
- **Scheduler cron** nội bộ, **self-healing** (tự tạo healer task retry ≤2 lần), **agent memory** lưu PostgreSQL (BM25 qua ParadeDB + pgvector cho semantic search).
- **Bảo mật:** không auth, không TLS — theo thiết kế, chỉ chạy trên mạng tin cậy; docs chỉ đích danh Tailscale/Headscale/Nebula làm lớp mã hóa bắt buộc cho "production".
## Yêu cầu triển khai
- Docker + Docker Compose; script `install.sh` dùng overlay `prod-local` → mở `http://localhost:8000`. Khởi tạo: PostgreSQL, NATS (port 8443), guacd, orchestrator (8000), **2 worker + 2 supervisor mặc định** (scale qua `--scale worker=4`).
- **Claude credentials (`~/.claude.json` + `~/.claude/.credentials.json`)** — tạo bằng cách chạy `claude` và đăng nhập (README nêu rõ case macOS với **Claude Code subscription**). Credential được bind-mount vào container worker/supervisor.
- Model mặc định: `SUPERVISOR_MODEL=claude-opus-4-6` (có thể override qua env, nhưng engine vẫn là Claude qua claude-agent-sdk).
- Optional: OpenAI API key (embedding memory, transcription patchright-cli — degrade sang BM25-only nếu không có), Telegram bot token (gateway).
- Tài nguyên: mỗi worker là cả desktop Linux + Chromium trong container — nặng, không hợp máy yếu.
## Docs có đủ để dựng không?
Có. README dài và chi tiết (quick start, install.sh, advanced setup với 3 compose overlay, env vars đầy đủ, thiết kế subject NATS, message flow, dev setup, testing). Thư mục `docs/` trong repo chỉ chứa ảnh (không có tài liệu chữ) — README là tài liệu chính. SECURITY.md liệt kê thẳng 19 trade-off bảo mật (no auth/TLS, bypassPermissions mặc định, VNC password broadcast plaintext qua NATS...) và ghi rõ phải chạy sau overlay mạng mã hóa.
## Verdict — Mạnh deploy được không?
Không, ít nhất là ở thời điểm hiện tại. Cơ chế multi-device rất đúng bài của Mạnh (NATS làm bus, worker attach/detach động trên máy vật lý qua VNC/SSH, Tailscale được docs nêu tên như lớp mạng chuẩn), nhưng figaro bị khóa cứng vào Claude: mọi worker/supervisor đều chạy Claude qua credential đăng nhập Claude Code (subscription trả phí), model mặc định là opus-4-6 — không có đường dùng model free (OpenRouter) đã xác minh, và mỗi worker ngốn cả một desktop Linux. Chừng nào Mạnh chưa có Claude subscription, repo này chỉ đáng tham khảo ý tưởng kiến trúc, không deploy được.
## Nguồn
- README gốc (kiến trúc, quick start, prerequisites, external desktops, config): https://github.com/byt3bl33d3r/figaro/blob/main/README.md
- SECURITY.md (no auth/TLS, Tailscale/Headscale/Nebula, bypassPermissions, credential bind-mount): https://github.com/byt3bl33d3r/figaro/blob/main/SECURITY.md
- Thư mục docs/ (chỉ chứa ảnh, không có file tài liệu): https://github.com/byt3bl33d3r/figaro/tree/main/docs
- Trang repo (stars 155, MIT, tác giả byt3bl33d3r): https://github.com/byt3bl33d3r/figaro
