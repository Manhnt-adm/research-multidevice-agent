# ai-maestro
- URL: https://github.com/23blocks-OS/ai-maestro
- Stars: 834 (mục "Repository details" trên trang GitHub, lấy ngày 23/10/2026)
- Đánh giá ngày: 23/10/2026

## Là gì (2 dòng)

"OS for AI-first organizations" — dashboard Next.js quản lý nhiều AI coding agent (Claude Code, Codex, Grok Build, OpenClaw...) chạy trong tmux session, có memory dài hạn, nhắn tin agent-to-agent và **multi-machine từ đầu**: peer mesh không server trung tâm. MIT license, không account, không telemetry.

## Kiến trúc multi-device

**Câu gốc README được xác minh:** "Multi-machine from day one — Peer mesh network with no central server. Nobody else does this."
(README, mục "What makes this different")

Cách mesh hoạt động, theo docs gốc:

- **Discovery (thủ công + lan truyền):** Trên mỗi máy đều cài AI Maestro đầy đủ. Thêm peer bằng tay trong Settings → Add Host, nhập URL `http://<peer-ip>:23000` (FAQ/SETUP-TUTORIAL khuyến nghị dùng IP Tailscale 100.x.x.x). Sau đó "Automatic Bidirectional Sync": thêm một lần từ bất kỳ node nào, hai bên tự discover nhau, và peer mới lan truyền tới tất cả node đã kết nối (eventually consistent).
  - Nguồn: https://github.com/23blocks-OS/ai-maestro/blob/main/docs/SETUP-TUTORIAL.md ("🔄 Automatic Bidirectional Sync! Add once from any node — both sides discover each other automatically. New peers propagate to all connected nodes.")
  - Nguồn: https://github.com/23blocks-OS/ai-maestro/blob/main/docs/CONCEPTS.md (mục "Automatic Peer Discovery": `POST /api/hosts/register-peer` + `POST /api/hosts/exchange-peers`)
- **Dashboard từ bất kỳ node nào:** Mọi node đều chạy full instance (không phân biệt master/worker), mở dashboard `:23000` ở bất kỳ node nào cũng thấy tất cả agent trên mọi máy. Session của peer đi qua proxy: Browser ↔ Node A ↔ (WebSocket qua Tailscale) ↔ Node B ↔ tmux local.
- **AMP là gì:** Agent Messaging Protocol — chuẩn mở (agentmessaging.org) cho agent nhắn tin kiểu email: có priority, message type, ký số Ed25519, push notification. Hai cách dùng: Skills Mode (nói tự nhiên với Claude Code, nó tự gọi amp-send) và Manual Mode (CLI `amp-send`/`amp-inbox`/`amp-read`/`amp-reply`, dùng được với mọi agent). Tin nhắn lưu file JSON tại `~/.agent-messaging/`. Cross-host: server nhận forward tin, verify chữ ký bằng key mà host gửi kèm (bad signature → 403; thiếu key → delivered nhưng ghi "unverified"). Liên đoàn với provider ngoài (vd CrabMail) qua `/api/v1/federation/deliver` với địa chỉ dạng `<name>@<org>.aimaestro.local`.
  - Nguồn: https://github.com/23blocks-OS/ai-maestro/blob/main/docs/AGENT-MESSAGING-GUIDE.md (1559 dòng, mục "Two Operational Modes", "Who sent it: signatures and trust")
- **Worker headless ~100MB:** Mỗi máy chạy headless (`yarn headless` / `yarn headless:prod`) là API-only, không Next.js/UI, phục vụ ~100 API endpoint qua standalone HTTP router (`services/headless-router.ts`), WebSocket (terminal, AMP, status) vẫn hoạt động như full mode. Startup ~1s, RAM ~100MB — "Ideal for worker nodes that only need the API surface". Dashboard đặt ở máy mình ngồi, agent chạy ở máy có compute.
  - Nguồn: https://github.com/23blocks-OS/ai-maestro/blob/main/docs/ARCHITECTURE.md (mục "Server Modes" → "Headless Mode")
- **Chuyển agent giữa các máy:** Agent = "nhân viên" sở hữu thứ của mình (repo, memory, schedule). Vì các máy không share filesystem, model đồng bộ duy nhất là git clone: "Transfer an agent to another host and AI Maestro clones its repositories to the same paths on the destination, because an agent's repos travel with the agent." Lịch hẹn (schedule) thuộc về agent, không thuộc máy, nên tự đi theo khi agent chuyển host.
  - Nguồn: https://github.com/23blocks-OS/ai-maestro/blob/main/docs/CONCEPTS.md (mục "Agent Ownership"), README mục "The Mental Model" và "Scheduled Work"
  - Ghi chú: chi tiết thao tác nút "move agent" ở UI/API cụ thể **chưa xác minh được** trong các docs đã đọc — chỉ thấy mô tả ở cấp concept.

## Yêu cầu triển khai

- **Cài trên mỗi máy (all peers):** Node.js 18.17+/20.x, tmux 3.0+, git; build-essential trên Linux (`sudo apt install tmux build-essential`); khuyến nghị Tailscale + pm2.
- **Nặng/lẻ:** Cần **4 GB RAM để build** (hoặc 2 GB + swap); sau khi build xong chạy trong "a few hundred MB", headless ~100MB. Machine đề xuất cho cloud VM: 4GB RAM, 2 vCPU, 15–25GB disk.
- **Platform:** macOS và Linux native; Windows qua WSL2 (Ubuntu 24.04 đã được contributor hỗ trợ). Dự án sinh ra từ giới Mac nhưng docs có scenario Cloud Server (AWS/DigitalOcean/Hetzner) đầy đủ.
- **Cài đặt:** one-line `curl ... | bash scripts/remote-install.sh` (~5–10 phút) hoặc clone + `yarn install && yarn build` + pm2.
- **Lưu ý bảo mật:** Mặc định KHÔNG có authentication trên API (dựa vào OS-level user security); docs khuyến nghị Tailscale để che giấu port 23000 khỏi internet công cộng. Không expose 23000 ra ngoài.
- Nguồn: README mục Quick Start; SETUP-TUTORIAL.md mục Prerequisites/Scenario 3; CONCEPTS.md mục Security Model; ARCHITECTURE.md mục Server Modes.

## Docs có đủ để dựng không?

**Đủ để dựng theo.** Bộ docs rất dày và thực dụng, không chỉ README sơ sài:
- `docs/SETUP-TUTORIAL.md` — step-by-step peer mesh (quick start 5 phút + 3 scenario: laptop+desktop Tailscale, LAN, cloud server + troubleshooting + best practices)
- `docs/CONCEPTS.md` — giải thích peer mesh, discovery, ownership, security model
- `docs/AGENT-MESSAGING-GUIDE.md` — 1559 dòng, hai mode dùng AMP
- `docs/ARCHITECTURE.md` — 1362 dòng, server modes, headless, test suites (kể cả cross-host mesh test qua Tailscale)
- `QUICKSTART.md`, Operations Guide, Network Access, Common Issues, Security — có liệt kê trong README (chưa đọc hết, chỉ ghi nhận tồn tại)

Thiếu/nhược điểm nhỏ: thao tác "move agent" giữa host cụ thể chưa thấy doc riêng; một số link docs trong README trỏ ra site ai-maestro.23blocks.com (chưa mở kiểm tra).

## Verdict — Mạnh deploy được không?

**Được về kỹ thuật, nhưng lệch use case và kẹt ở "não" agent.** Mesh Tailscale + headless 100MB rất hợp hạ tầng của Mạnh (VPS + Termux, Tailscale đã có sẵn; headless chạy ngon trên VPS yếu), MIT + không telemetry hợp "quy chế vận hành". Vấn đề: repo được thiết kế cho **Claude Code / Codex / Grok Build — đều là subscription trả phí** (FAQ: "You bring your own agent subscriptions"), Mạnh không có subscription trả phí. Agent "bất kỳ chạy trong terminal" vẫn chạy được trong tab terminal, nhưng hỗ trợ first-class (launch, chat, approval card, inbox mod) chỉ dành cho 3 tool trả phí; ZeroClaw/OpenRouter-free của Mạnh sẽ chỉ ở mức "terminal tab" chưa được docs xác nhận. Thêm nữa build cần 4GB RAM (VPS Oracle 1GB sẽ phải dùng swap, hơi cực). Kết luận: nếu Mạnh chấp nhận chạy agent dạng terminal-generic thì đáng thử PoC trên 1 VPS; nếu muốn đúng như demo của repo thì phải có Claude Code subscription — chưa phù hợp lúc này.

## Nguồn
- https://github.com/23blocks-OS/ai-maestro (README — câu "Multi-machine from day one — Peer mesh network with no central server", Quick Start, Features, FAQ)
- https://github.com/23blocks-OS/ai-maestro/blob/main/docs/SETUP-TUTORIAL.md (Peer Mesh Setup Tutorial — discovery, 3 scenario, troubleshooting)
- https://github.com/23blocks-OS/ai-maestro/blob/main/docs/CONCEPTS.md (peer mesh, automatic peer discovery, agent ownership/transfer, security model)
- https://github.com/23blocks-OS/ai-maestro/blob/main/docs/AGENT-MESSAGING-GUIDE.md (AMP: 2 operational modes, signatures, storage, federation)
- https://github.com/23blocks-OS/ai-maestro/blob/main/docs/ARCHITECTURE.md (Server Modes: Headless ~100MB; AMP cross-host mesh tests via Tailscale)
