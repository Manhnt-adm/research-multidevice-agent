# Muxin
- URL: repo https://github.com/LumaryLabsLLC/muxin + website https://muxin.dev/
- Stars: 362 (tại thời điểm đọc trang repo)
- Đánh giá ngày: 23/10/2026

## Là gì (2 dòng)

Muxin là desktop app viết bằng Rust (GPUI, framework của Zed editor) — một "hub" gom mọi máy tính của bạn vào một cửa sổ: chạy session agent AI trên bất kỳ máy nào, xem màn hình live, đọc log, quản lý Docker/GitHub/Schedules/VMs, copy file giữa các máy.
Điểm đặc biệt: mọi tính năng đều phơi ra cho agent AI thông qua MCP server 89 tools và CLI (`muxin`), để agent "dùng" được toàn bộ dàn máy như con người.

## Kiến trúc multi-device

- **Kết nối đầu tiên = SSH.** Muxin chạy trên máy của bạn (macOS/Linux/Windows), kết nối tới từng máy khác bằng chính SSH của hệ thống: key, ssh agent, và `~/.ssh/config` bạn đã có. "Không có account, không có server của hãng ở giữa" (https://muxin.dev/docs).
- **Helper agent `muxen-agent` trên từng máy.** Lần đầu dùng một máy cho việc gì hơn "check nhanh", Muxin tự copy agent nhỏ của nó (vài MB) vào `~/.muxen` trên máy đó qua SSH, tự giữ agent cập nhật. Từ đó Muxin nói chuyện với agent qua **một kết nối ssh duy trì** (giữ mở, tự reconnect) thay vì spawn ssh mới mỗi request.
- **Direct pairing (tùy chọn):** "Connect directly" trên trang máy — hai bên trao đổi certificate qua ssh, agent chạy listener như user service (systemd/launchd) trên UDP port 47821, app nối tới qua QUIC, fallback về ssh khi direct rớt.
- **Tailscale là lớp reachability.** Muxin không config Tailscale mà chỉ đọc status của nó: máy đã pair được thử qua địa chỉ local trước, Tailscale sau; máy nào host chỉ có IP local thì ssh tự đổi sang tailnet address khi IP thường không đáp. Có nút "Set up Tailscale" gợi ý cài trên máy Linux chưa có.
- **Session persistent (kiểu tmux).** `muxen-agent` (ở `~/.muxen/bin/muxen-agent`) là background program giữ terminal của mỗi session. Đóng app, sleep laptop, mất mạng — chỉ detach; agent tiếp tục chạy; mở lại thì thấy màn hình hiện tại + scrollback. **Khởi động lại máy thì session mất** (giống tmux). Có thể detach/reattach session từ CLI: `muxin session attach <id>`.
- **Agent "borrow" máy = Agent Workspaces.** Agent xin một workspace: folder persistent trên một máy (`~/muxen/workspaces/<id>`), tùy chọn clone repo ở branch/ref, chạy lệnh trong môi trường login của ssh user, đọc/ghi file, review git diff, sync 2 chiều với checkout local qua rsync, hoặc chạy trong container Docker cô lập (mặc định `buildpack-deps:bookworm`, mount `/workspace`), hoặc worktree của checkout sẵn có để nhiều agent cùng làm một repo song song. Muxin có thể tự chọn máy "least-loaded online".
- **MCP server 89 tools** (`muxin mcp`, stdio JSON): status/check máy, session_* (start/read/send/question/answer/action/stop), workspace_*, logs/services/Docker/Compose, processes/ports, desktop_screenshot/desktop_input, GitHub (inbox/repos/issues → giao issue cho agent), runners, VMs, schedules, cloud machines, usage (thống kê token/tiền của Claude Code/Codex theo API prices), và long operations. Mọi lệnh CLI đều có `--json` cho agent không nói được MCP.
- **Peer-to-peer sharing:** share session/máy qua link `https://muxin.dev/s#...` (WebRTC P2P, mã hóa; muxin.dev chỉ làm "môi giới" 2 message, secret sau `#` không tới site), guest xem/browser hoặc Muxin, có cơ chế 6-digit code đọc to để admit.

## Yêu cầu triển khai

- **App/desktop:** macOS 12+ (.dmg, signed/notarized), Windows 11 (.exe — chưa signed), Linux glibc ≥2.35 (.deb/.rpm/.pkg.tar.zst/AppImage/tar.gz không cần root/AUR). Có bản headless cho server: `cargo build --release --no-default-features` → chỉ CLI + MCP server (không GUI).
- **Agent CLI được hỗ trợ:** Claude Code (`claude`), Codex (`codex`), OpenCode (`opencode`), Gemini CLI (`gemini`), Amp (`amp`), Pi — hoặc plain shell. App tự phát hiện CLI nào đã có trên từng máy. Session chạy trong login shell nên nvm/asdf và login của các CLI hoạt động như trong terminal thật.
- **Subscription trả phí — ĐIỂM QUAN TRỌNG:** Muxin bản thân **miễn phí, không cần subscription của nó**. Nhưng các session agent nó chạy là **các CLI bạn tự mang tới**: README có trang "Usage" thống kê "what Claude Code and Codex used" và ghi rõ "With a Claude or ChatGPT plan you don't pay per token" — tức thực tế muốn chạy Claude Code/Codex cần plan của Anthropic/OpenAI (hoặc API key có tiền). OpenCode là BYOK (cần API key bạn tự nạp). Gemini CLI có free tier. Muxin không chạy agent free nào của riêng nó; với Mạnh (không có subscription trả phí) chỉ còn đường dùng CLI có free tier/BYOK (Gemini CLI free, hoặc OpenCode + key free của chính anh).
- **Mạng:** SSH reach được (LAN, Internet, hoặc Tailscale). Muxin đọc `~/.ssh/config`, tailnet, Bonjour để gợi ý máy. Không yêu cầu mở firewall thêm cho trường hợp cơ bản; direct connection cần UDP 47821 tới được máy đích.
- **GitHub (tùy chọn):** `gh auth login` hoặc token; runners org cần scope `admin:org`. Cloud machines (tùy chọn): token DigitalOcean, máy do bạn trả tiền trực tiếp cho DigitalOcean.

## Docs có đủ để dựng không?

**Đủ để dựng.** Không chỉ marketing: website có đầy đủ trang docs (https://muxin.dev/docs + https://muxin.dev/docs/agents), README trên repo dài chi tiết (Install/Quick start/config.toml/CLI/MCP/workspace/session/Tailscale/Sharing/Cloud/VMs/Build from source), kèm ví dụ lệnh cụ thể và schema lỗi JSON cho agent. Có cả `muxin --demo` để khám phá bằng máy mô phỏng trước khi đụng máy thật. Thiếu sót duy nhất: dự án rất mới (repo tạo 08/10/2026, mới ~95 commit, version 0.6.0) — docs có thể chưa bao hết edge case thực tế.

## Verdict — Mạnh deploy được không?

**Được, với điều kiện kiểm soát kỳ vọng — xếp loại "khó vừa" với stack của Mạnh.** Ưu: free/open source Apache 2.0; dùng đúng SSH+Tailscale mesh anh đã có; helper agent tự cài qua SSH nên S8+/Termux/VPS chỉ cần SSH chạy là nhận (lưu ý SSH trên Termux/phone qua Tailscale OK, nhưng session agent cần máy duy trì background — phone tắt màn kill process có thể là vấn đề, chưa xác minh); có bản headless CLI+MCP chạy được trên VPS không GUI. **Rủi ro chính:** (1) Muxin không có agent free đi kèm — mọi session thông minh đều qua CLI ngoài mà Claude Code/Codex đòi plan trả phí, chỉ Gemini CLI free tier và OpenCode BYOK là khả thi với anh; (2) đây là "hub" kiểu người điều khiển — khác với conductor/worker tự động chạy 24/7, nó không sinh ra agent chạy độc lập mà là nơi agent sống; (3) dự án mới 2 tuần tuổi (08/10/2026, 362 stars), chưa được chứng minh dài hạn. Nếu track multi-device của anh muốn agent tự hành đa máy giá $0: Muxin làm tốt lớp "điều phối + truy cập máy", nhưng lớp "não agent free" vẫn phải tự lo (Gemini CLI free / OpenCode + OpenRouter key của anh).

## Nguồn
- Website chính: https://muxin.dev/ (tổng quan tính năng, MCP 80+ tools, CLI ví dụ, yêu cầu platform, FAQ giá/license, Homebrew tap `lumarylabsllc/tap`)
- Docs kiến trúc: https://muxin.dev/docs (SSH → muxen-agent, direct QUIC, Tailscale, sessions persistent)
- Docs MCP: https://muxin.dev/docs/agents (89 tools, setup cho Claude Code/Codex/MCP clients, danh sách tools theo nhóm)
- Repo + README đầy đủ: https://github.com/LumaryLabsLLC/muxin (362 stars, 95 commits, tạo 08/10/2026, license Apache 2.0; mục Install/Quick start/config.toml/Agent workspaces/Agent sessions/Connections & desktops/Tailscale/headless build `cargo build --release --no-default-features`/Usage ghi "With a Claude or ChatGPT plan you don't pay per token"/sharing P2P)
