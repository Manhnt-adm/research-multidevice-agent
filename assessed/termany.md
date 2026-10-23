# termany

- URL: https://github.com/thinkany-ai/termany
- Stars: 486 (tại thời điểm đọc)
- Đánh giá ngày: 23/10/2026

## Là gì (2 dòng)

Terminal app (Tauri desktop + Node PTY server) gom nhiều coding-agent session vào một cửa sổ: pane lồng 3 cấp (workspace → page → tab), gắn tag working/done/needs-attention cho từng pane.
Mỗi pane SSH riêng tới remote host; có thêm "Bots" (agent wrapper qua ACP) chat nhóm với nhau trong app.

## Kiến trúc multi-device (SSH như thế nào, có giao việc/thu kết quả tự động không hay chỉ là TUI cho người)

**SSH — per-pane, tương tác, không có orchestration.** Mỗi pane tự chọn connection riêng (display name, port, auth tự động/password/identity-file), có nút Test connection, rớt mạng thì Enter để reconnect; khi remote pane serve port thì 1 click mở SSH forward về loopback. Tất cả phục vụ **người ngồi nhìn và gõ** — không có CLI headless, không API, không script điều khiển session, không task queue.
- Nguồn: https://github.com/thinkany-ai/termany (README mục "Worktrees and remote hosts": "Per-pane SSH. Each pane picks its own connection...")

**"Bots that work together" — delegation trong chat, không phải orchestration máy.** Người dùng tạo Bot (name, avatar, role, model, working folder) từ agent CLI (Claude, Codex, Gemini, Grok Build, OpenClaw, Hermes, OpenCode, Cursor, Kimi, OMP...), cho nhiều Bot vào group chat: một lead member điều phối hội thoại, delegate việc, consolidate kết quả; `@` mention route message. Đây là chat giữa các agent **trong cùng app, tương tác**, không phải dispatch task ra nhiều máy và thu kết quả tự động. `docs/` trong repo chỉ chứa screenshot và tài liệu bot-skills — không có tài liệu automation/API nào.
- Nguồn: https://github.com/thinkany-ai/termany (README mục "Bots that work together"); https://github.com/thinkany-ai/termany/tree/dev/docs (chỉ screenshot + bot-skill docs)

**Kết luận kiến trúc:** đây là **multiplexer cho người nhìn** (terminal nhiều pane + SSH), không phải hệ điều phối multi-device.

## Yêu cầu triển khai

- Tải build từ https://termany.sh (macOS `.dmg` ký + notarize; Windows `.exe` NSIS unsigned) — **không có bản Linux server/headless**.
- Hoặc chạy từ source: `npm install` (build node-pty native, cần Xcode CLT trên macOS) → `pnpm dev:web` (PTY server :5175 + web :15173) hoặc `pnpm dev:desktop` (Tauri).
- Model providers theo kiểu BYOK (bring your own key), cấu hình trong app; Bot dùng CLI đã cài + authenticate sẵn trên máy local (Claude, Codex...).

## Docs có đủ để dựng không?

Đủ cho **dùng tương tác** (README dài, chi tiết từng tính năng, có website termany.sh với downloads/docs/release notes). Nhưng **không có tài liệu nào cho automation/orchestration** vì sản phẩm không có tính năng đó — không phải thiếu docs mà là thiếu tính năng.

## Verdict — Mạnh deploy được không?

**Loại** (cho use case điều phối fleet).

1. Chỉ là multiplexer cho người nhìn: SSH per-pane tương tác, không giao việc/thu kết quả tự động, không headless CLI/API — không đáp ứng định nghĩa "điều khiển nhiều máy" của track.
2. App desktop macOS/Windows, không chạy headless trên VPS/A1 Flex được; group-bot delegation chỉ là chat trong app, không dispatch được ra S8+/S20FE qua Tailscale.

Lưu ý phụ: license AGPLv3 (+ commercial từ ThinkAny, LLC) — dùng cá nhân không modify thì không vấn đề gì, nhưng cũng không liên quan vì đã loại. Không phù hợp làm conductor; tối đa chỉ là công cụ quan sát thủ công nếu Mạnh ngồi máy desktop.

## Nguồn (URL cụ thể đã đọc)

- https://github.com/thinkany-ai/termany (README đầy đủ: per-pane SSH, Bots group chat, install/run-from-source, BYOK; repo metadata: 486★, tạo 2026-06-29, default branch dev, 232 commit, 31 release)
- https://github.com/thinkany-ai/termany/tree/dev/docs (chỉ screenshot + tài liệu bot-skills, không có automation docs)
- https://github.com/thinkany-ai/termany/blob/dev/LICENSE (AGPLv3, commercial license liên hệ support@thinkany.ai)
