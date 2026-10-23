# Cordane
- URL: https://github.com/cordane/cordane
- Stars: 3
- Đánh giá ngày: 23/10/2026

## Là gì (2 dòng)
Cordane là **control plane tự host cho AI coding agents** (Claude Code, Codex, opencode, pi): một bảng kanban web biến ticket thành git worktree + agent chạy trên máy của bạn, kèm terminal thật (PTY) truy cập từ bất kỳ browser nào. Điểm nhấn: worker **chỉ dial-out** về hub, không cần mở port inbound/VPN/SSH.

## Kiến trúc multi-device
- **Hub** (control plane) + **worker** là hai vai của **một binary Go duy nhất** (`cordane server` / `cordane worker`).
- **Giao thức worker→hub**: worker mở `GET /api/v1/workers/connect` với bearer token, upgrade thành **WebSocket outbound duy nhất**. Trên WebSocket này chạy **yamux multiplexing**: stream đầu tiên (do worker mở) là **control channel** (newline-delimited JSON: request có `id`, response echo `id`, event không có `id`); các stream sau do server mở, bắt đầu bằng header JSON một dòng `{"kind":"pty","session_id":"…"}` (PTY bytes thô) hoặc `{"kind":"proxy","app_id":"…"}` (pipe TCP thô tới port local của worker cho app preview/dev server).
- **Join worker**: `cordane worker join https://<hub> --token crdjt_…` (token một lần lấy từ trang Workers trên hub); worker enroll và chạy ngay, giữ kết nối sống, tự reconnect. Để chạy nền qua reboot: `cordane worker run` dưới systemd user unit hoặc launchd.
- **Handshake chống version skew**: message đầu là `hello` kèm protocol version, build string, capability list; hub trả `server.hello` (cố tình không có `id` để worker cũ bỏ qua). Hub **refuse thẳng** worker dưới protocol floor; tính năng mới chỉ bật khi nằm ở **giao nhau của capability hai bên**.
- **State nằm trên worker** (PTYs, worktree, port reservation — file worker-side), hub chỉ là relay/coordinator; restart hub không giết terminal. Hub state = **SQLite**, backup bằng **Litestream** replicate WAL tới bucket S3-compatible.
- Task format: "ticket" trên board → hub cắt **git worktree** trên branch mới từ main, reserve block port riêng, chạy agent headless theo plan/implement prompt; diff + check hooks quay về card để review, có review rounds và race nhiều agent.

## Yêu cầu triển khai
- **Hub**: box Linux có Docker, 2 container (caddy + cordane qua docker-compose), ~**1 vCPU / 512MB–1GB RAM**, vài GB disk, cần **tên miền + record A** (TLS tự động qua Caddy), và **bắt buộc tạo GitHub OAuth App** (sign-in chỉ qua GitHub, không có password; tài khoản sign-in đầu tiên = admin). Prebuilt image từ GHCR, không cần compile gì.
- **Worker**: Linux hoặc macOS (install.sh hỗ trợ linux/darwin × amd64/arm64); **git bắt buộc**; Docker tùy chọn (sandbox); phải tự cài và **sign-in sẵn agent CLI** trên worker (claude, codex, opencode, pi) — Cordane không mang theo agent/login, "drives the agent that's there".
- Nhẹ về tài nguyên, nhưng **không mã nguồn mở phần mềm**: repo GitHub chỉ chứa README + chính sách + config deploy (MIT); image `ghcr.io/cordane/cordane` là **proprietary của Sitedity SRL** (ghi rõ trong NOTICE). **Bản Community free giới hạn: 1 worker, 1 project, 2 user, 3 concurrent agent runs** — vượt là phải mua tier trả phí.

## Docs có đủ để dựng không?
Đủ để dựng theo: README chi tiết, docs site https://cordane.ai/docs (quickstart self-host 3 bước, connect worker, GitHub OAuth), deploy/README.md đầy đủ (.env, wildcard vs simple mode, systemd unit, update/rollback), và bài blog kiến trúc "One outbound WebSocket" giải thích protocol yamux + handshake + security posture. Thiếu: không có source code để đọc sâu hơn (proprietary), không có tài liệu về việc chạy worker trên Android/Termux.

## Verdict — Mạnh deploy được không?
**KHÔNG phù hợp, bỏ qua.** Ba lý do cứng: (1) Cordane là công cụ **chạy coding agent** (Claude Code/Codex...) — yêu cầu subscription AI trả phí trên từng worker, trong khi Mạnh không có subscription nào; (2) **không phải open source** — app là image proprietary, không nghiên cứu/tùy biến được cho mục đích rebuild conductor; (3) **free tier giới hạn 1 worker** — đúng với use case multi-device (VPS + nhiều điện thoại) là phải trả tiền, trong khi có nhiều framework orchestration open-source thật sự không giới hạn. Điểm đáng học hỏi duy nhất: pattern **worker dial-out + yamux multiplexing trên một WebSocket** là thiết kế NAT-traversal rất gọn, đáng tham khảo cho blueprint.

## Nguồn
- README repo: https://github.com/cordane/cordane (stars: 3, tạo 2026-09-01)
- NOTICE (repo chỉ chứa docs + deploy config; image proprietary của Sitedity SRL; Community tier 1 worker/1 project/2 users/3 concurrent runs): https://github.com/cordane/cordane/blob/main/NOTICE
- Docs install & run (quickstart, connect worker, GitHub OAuth, yêu cầu worker): https://cordane.ai/docs
- deploy/README.md (docker-compose, .env, systemd unit cho worker): https://github.com/cordane/cordane/blob/main/deploy/README.md
- Blog kiến trúc "One outbound WebSocket" (protocol yamux, handshake hello/capabilities, stream header JSON, control channel): https://cordane.ai/blog/outbound-websocket
- install.sh (hỗ trợ linux/darwin × amd64/arm64): https://cordane.ai/install.sh
- Trang chủ (hub ~1 vCPU 512MB–1GB RAM; worker dial-out): https://cordane.ai/
