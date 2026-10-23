# zerus
- URL: https://github.com/ufna/zerus
- Stars: 14 (tại thời điểm đọc)
- Đánh giá ngày: 23/10/2026

## Là gì (2 dòng)
Zerus là ADE (Agent Development Environment): desktop app (Qt) + CLI `hgs` quản lý một "fleet" các coding-agent session (Claude Code, Codex, Kimi Code, DeepSeek Harness, OpenCode) chạy native trong tmux trên nhiều máy. Giấy phép MIT; repo mới 16 ngày tuổi (tạo 2026-10-07), 2.377 commit / 55 release, gần như một mình tác giả Vladimir (@ufna) đẩy.

## Kiến trúc multi-device (fleet across machines + P2P sync chi tiết ra sao)
- **Mô hình peer, không máy chủ trung tâm:** "your machines are peers in one shared workspace. A project can include folders and agent sessions on several hosts. The project catalog, session assignments and shared recovery settings sync directly between machines, with no permanent main machine." (README)
- **"P2P" thực chất = sync qua SSH, KHÔNG phải libp2p/iROH:** bảng Features ghi rõ "sync peer to peer over SSH. Each machine keeps a copy... Offline peers catch up when they reconnect; SSH access, credentials and appearance stay local." Kiểm tra trong reference.md không có chuỗi "libp2p". Cơ chế swarm dùng `hgs swarm hello/export/get` và shared "swarm catalog" replicate giữa các member (`swarm/mobile-peers.json`).
- **Quan trọng: sync chỉ metadata, không sync file/code:** "project synchronization does not move conversations" và "The folders and agents stay on their hosts; the project organization and shared settings follow you between connected computers." — tức là catalog dự án + gán session + setting recovery được đồng bộ, còn folder code và lịch sử conversation nằm yên trên host gốc.
- **Agent chạy ở đâu:** session chạy trên máy nào cài native harness, trong tmux trên máy đó; điều khiển máy khác qua `hgs @host <agent> <project>` (SSH). Máy điều khiển có thể là Windows "control console" (không chạy session).
- **Orchestration:** có thật nhưng ở dạng coordination primitives do người/agent gọi chủ động — `hgs send` / `hgs inspect` / `hgs wait` (khai báo dependency giữa các session, ticket-based ack/finish, journal SQLite, hỗ trợ cross-machine qua `hgs @host`). Đây là message-passing + dependency tracking có kiểm chứng, không phải scheduler tự động phân task.
- **Điện thoại là client, không phải host:** Android APK / iOS (TestFlight beta) kết nối tới máy tính qua HTTPS relay tự host được (outbound HTTPS, không cần mở port/VPN); phone đọc snapshot, gửi lệnh, nhận push qua FCM/APNs. Docs không đề cập việc biến Android thành peer host chạy agent (không có build Termux/host Android).

## Yêu cầu triển khai
- **CLI `hgs`:** Rust stable mới nhất, C compiler, Python 3, **tmux 3.7+** (bắt buộc cho verified message delivery), OpenSSH; cài thêm agent muốn dùng trên từng máy chạy session. (docs/installation.md)
- **Cài đặt:** `git clone` + `./install.sh` (+ `./tray/install.sh` cho desktop). Prebuilt chỉ có Arch x86_64 (AUR), macOS (DMG/Homebrew), Windows (portable ZIP). **Ubuntu/Debian → phải build từ source bằng Rust.**
- **Desktop GUI:** CMake, C++17, Qt 6 + KDE Frameworks 6, systemd autostart — nặng, chỉ hợp máy desktop/laptop, không cần cho headless.
- **Mobile relay (tự host):** Rust runtime, SQLite cho local/dev, PostgreSQL cho production; cần TLS + FCM (Android)/APNs (iOS) cho push.
- **Harness được hỗ trợ đầy đủ:** Claude Code, Codex, Kimi Code, DeepSeek Harness, OpenCode. "You can run any terminal command through hgs; full ADE features need an adapter." Model local qua Ollama + adapter Codex/OpenCode (docs/local-models.md). Không có doc nào nhắc OpenRouter.

## Docs có đủ để dựng không? (repo 14 sao — đánh giá trung thực độ chín)
Docs rất dày và chi tiết hiếm thấy ở repo 14 sao: reference.md ~6.475 dòng, installation/mobile-architecture/relay-architecture/agent-coordination/local-models đều viết kỹ, có CI + nightly + AUR + checksum release. Độ chín kỹ thuật của docs cao. Nhưng: repo mới 16 ngày, một maintainer, API/CLI còn biến động nhanh (55 release trong 16 ngày) — rủi ro breaking change và bug chưa lộ. Không có doc cho Termux/Android-host hay OpenRouter.

## Verdict — Mạnh deploy được không?
**Được có điều kiện.** SSH-peer + Tailscale mesh rất hợp hạ tầng của Mạnh; CLI Rust nhẹ, chạy CLI-only (không cần desktop Qt) trên A1 Flex 12GB là thoải mái, VPS Oracle 1GB cũng đủ cho `hgs`+tmux (nhưng phải build Rust từ source — trên 1GB RAM dễ OOM/chậm, nên build ở A1 rồi copy binary hoặc cross-compile). Điều kiện: (1) Zerus chỉ quản lý **coding agents** trong danh sách hỗ trợ — Antigravity Pro (IDE, không API) của Mạnh KHÔNG được support, chỉ chạy được như "terminal command" thô không full features; đường đi khả thi nhất là OpenCode/Codex + model free qua OpenRouter (chưa xác minh qua docs). (2) 2 điện thoại chỉ làm **client theo dõi**, không thành peer host chạy agent được — không thay thế được "tay ADB" trong kiến trúc 3 tầng. (3) "P2P sync" chỉ đồng bộ catalog/session metadata, không sync code hay conversation. Không thay thế ZeroClaw cho automation tổng quát.

## Nguồn (URL cụ thể đã đọc)
- https://github.com/ufna/zerus (README: định nghĩa ADE, fleet/P2P sync over SSH, agent-to-agent comms, quick start; repo details: 14 stars, created 2026-10-07, 2.377 commits, 55 releases, MIT)
- https://github.com/ufna/zerus/tree/main/docs (danh sách docs/)
- https://raw.githubusercontent.com/ufna/zerus/main/docs/installation.md (yêu cầu CLI/desktop, prebuilt chỉ Arch/macOS/Windows, Ubuntu build từ source)
- https://raw.githubusercontent.com/ufna/zerus/main/docs/reference.md (CLI sessions, swarm join, "project synchronization does not move conversations", Machines/accounts)
- https://raw.githubusercontent.com/ufna/zerus/main/docs/agent-coordination.md (hgs wait/send/inspect, ticket-based dependency protocol)
- https://raw.githubusercontent.com/ufna/zerus/main/docs/mobile-architecture.md (phone = client qua relay HTTPS tự host, swarm catalog, one-hop boundary)
- https://raw.githubusercontent.com/ufna/zerus/main/docs/relay-architecture.md (relay Rust, SQLite local / PostgreSQL production, FCM/APNs)
- https://raw.githubusercontent.com/ufna/zerus/main/docs/local-models.md (tách harness vs inference provider, Ollama + Codex/OpenCode adapter)
