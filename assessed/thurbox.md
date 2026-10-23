# thurbox

- URL: https://github.com/Thurbeen/thurbox
- Stars: 112 (tại thời điểm đọc)
- Đánh giá ngày: 23/10/2026

## Là gì (2 dòng)

TUI viết bằng Rust để chạy nhiều coding agent song song trong một terminal, mỗi agent sống trong một tmux session persistent (sống sót crash/restart/reboot).
Hỗ trợ session trên remote host qua SSH (khai báo trong `hosts.toml`), session hiện cùng danh sách với session local; kèm CLI headless `thurbox-cli` để điều khiển session bằng script.

## Kiến trúc multi-device (SSH như thế nào, có giao việc/thu kết quả tự động không hay chỉ là TUI cho người)

**SSH:** mỗi host khai báo trong `hosts.toml`; cùng một "session backend" adapter (tmux/psmux) chạy trên tmux server của host đó qua SSH/WSL (theo ADR-13 trong `docs/ARCHITECTURE.md`). TUI vẫn chạy local, session remote hiện dưới host của nó. Remote session cũng báo trạng thái (working/blocked/done) qua hook provisioning lên host, nhận về qua control-mode connection của tmux.
- Nguồn: https://github.com/Thurbeen/thurbox (README: "Agents on other machines too — sessions on a remote host over SSH... declare hosts in hosts.toml"); https://github.com/Thurbeen/thurbox/blob/main/docs/ARCHITECTURE.md (ADR-2: adapters run over SSH or WSL)

**Orchestration — CÓ, không chỉ là multiplexer cho người nhìn.** Đây là điểm thurbox khác biệt:
1. `thurbox-cli` headless: tạo, điều khiển (gửi text vào session, `session send --wait-turn`), chụp màn hình (`session capture`), chờ trạng thái (`session wait --state done,blocked`), đóng session — tất cả từ script, đọc cùng database với TUI.
2. Inter-session messages: mailbox agent-neutral để một agent gửi payload cho agent khác (`thurbox-cli message send --to <lead> --kind result`), không phải scrape terminal.
3. Lead/worker trees: spawn worker với `--parent <uuid>` của lead, liệt kê được cả worker chưa bao giờ báo cáo.
4. Automations/tasks: chạy agent theo lịch (tmux heartbeat 60s kích hoạt cả khi TUI đóng).
5. Extension **Fleet** (pattern control-plane được maintain): một lead session "Mission Control" biến goal thành queue task (`queue topic add` → `queue add` tạo `BRIEF.md` mỗi task) → `queue dispatch` phóng tất cả worker song song, mỗi worker một git worktree → `queue watch`/`queue collect` đọc `result.md` worker viết và kiểm tra artifact → `queue reap` dọn session.
- Nguồn: https://github.com/Thurbeen/thurbox/blob/main/docs/ORCHESTRATION.md (toàn bộ run loop, dòng ~122–150; report-by-file, mailbox, --parent, reconciler)

**Giới hạn cần thấy rõ:** worker vẫn là một coding-agent CLI tương tác (Claude Code, Codex, opencode, aider...); "giao việc" = gửi prompt vào pane, "thu kết quả" = đọc `result.md`/pane output. Không phải executor chạy lệnh thô (shell command) trả exit code như conductor của blueprint.

## Yêu cầu triển khai

- `thurbox` + `thurbox-cli` qua `scripts/install.sh` (checksum, auto-detect platform), hoặc nix/homebrew/AUR/winget, hoặc build từ source.
- Bắt buộc: tmux ≥ 3.2 (hoặc psmux ≥ 3.3.8 trên Windows, RMUX ≥ 0.10.0 opt-in), git, ít nhất một coding-agent CLI.
- Remote host: SSH tới được, có tmux + coding CLI trên host đó; khai báo trong `hosts.toml`.
- Binary Rust nên rất nhẹ — chạy được trên máy yếu; SQLite local làm database.

## Docs có đủ để dựng không?

Đủ và dư. README có quickstart 4 bước; manual đầy đủ tại https://thurbox.thurbeen.eu/docs (installation, tutorial, features, keybindings, configuration, agents, orchestration, architecture, so sánh với tool tương tự, FAQ); rationale trong repo `docs/` (CONSTITUTION, ARCHITECTURE, FEATURES, CONFIG, KERNEL, PLUGINS, ORCHESTRATION, DEVELOPMENT, RELEASING). 1690 commit, CI chạy.

## Verdict — Mạnh deploy được không?

**Được có điều kiện.**

1. Orchestration là thật (programmatic CLI + lead/worker + task queue + thu result), không phải multiplexer thuần — đáp ứng tiêu chí "điều khiển nhiều máy" của track, và binary Rust đủ nhẹ cho VPS Oracle 1GB.
2. Nhưng thiết kế quanh **coding-agent CLI tương tác**, không phải remote command executor — không thay thế được conductor REST trong blueprint (claim atomic, evidence check, gates) cho tác vụ machine-to-machine.
3. Repo còn rất mới (tạo 02/2026, 112★, niche) — rủi ro đổi API; README liệt kê Antigravity trong danh sách agent được hỗ trợ nhưng cách Antigravity (IDE, không API) chạy như một CLI trong pane **chưa xác minh được**.

Phù hợp nếu Mạnh muốn một "bảng điều khiển + dispatcher" cho coding agents trên nhiều máy qua SSH/Tailscale (ví dụ: phóng N agent code song song trên A1 Flex + VPS); không phù hợp làm conductor chính của fleet.

## Nguồn (URL cụ thể đã đọc)

- https://github.com/Thurbeen/thurbox (README + repo metadata: 112★, MIT, tạo 2026-02-12, 1690 commit)
- https://github.com/Thurbeen/thurbox/blob/main/docs/ORCHESTRATION.md (control-plane pattern, Fleet run loop, mailbox, --parent, watch/collect)
- https://github.com/Thurbeen/thurbox/blob/main/docs/ARCHITECTURE.md (ADR-2 session backend qua SSH/WSL, tmux persistent)
- https://github.com/Thurbeen/thurbox/blob/main/docs/FEATURES.md (remote session status hooks, hosts, lifecycle)
