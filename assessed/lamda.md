# lamda
- URL: https://github.com/firerpa/lamda
- Stars: 8647 (ghi trên trang GitHub repo; forks 1167; license MIT; 64 releases; tạo 2022-07-10 — repo sống 6+ năm)
- Đánh giá ngày: 23/10/2026

## Là gì (2 dòng)
FIRERPA/lamda là nền tảng điều khiển Android full-stack theo mô hình client/server: server chạy trực tiếp trên máy Android (không cần runtime phụ, chạy được root hoặc không root/Shizuku), phía PC điều khiển qua thư viện Python, remote desktop trên trình duyệt (WebRTC/H.264/MJPEG, tối đa 60fps), 160+ API Python, Frida tích hợp sẵn, MITM một chạm, proxy/VPN/frp/P2P, MCP server và lệnh `agent` điều khiển bằng ngôn ngữ tự nhiên — tự định vị là "superset" của Appium + mitmproxy + frida-server + adb + uiautomator2 trong một pipeline.

## Kiến trúc multi-device
- **Mô hình C/S tập trung**: server lamda trên mỗi máy (một cổng duy nhất phục vụ remote desktop, RPC, terminal, file, MCP), client Python orchestrate từ PC — kiến trúc này "tiện cho centralized scheduling, versioning, fleet control" hơn script chạy on-device (kiểu AutoJS), nhẹ hơn Appium, và docs khẳng định ổn định hơn uiautomator2 trong kịch bản multi-device.
- **Pigeon platform (Pigeon + hub-bridge)** là lớp quản trị tập trung: gom các máy local + remote vào một dashboard, truy cập **P2P** ("máy ở nhà, người ở nơi khác", multi-machine collaboration); hub v3 dùng **SAPI** để gọi thống nhất tới máy local và remote. Repo hub đã đổi tên thành **firerpa/pigeon** (mới: 46 stars, 8 commit) — còn thô sơ.
- **P2P Bridge peer-to-peer** và **hệ thống task phân tán built-in** có từ version 9.0 (changelog); cơ chế chi tiết bên trong (giao thức, hàng đợi, retry, supervisor) chưa xác minh được từ docs.
- **Self-host không qua vendor**: docs nêu rõ có thể dùng **frp forwarding** hoặc **OpenVPN mesh** tự dựng (có docs + Docker image/config generator đi kèm) thay cho dịch vụ mạng của vendor ("networking subscription service" từ v7.50, cho phép networking không cần frp/OpenVPN — có/không trả phí chưa xác minh được, nhưng docs khẳng định self-host là phương án thay thế đầy đủ).
- **mDNS discovery** (`{device_id}.local` + metadata TXT) giúp phát hiện hàng loạt máy trong LAN.
- Mỗi máy có **Virtual Display** (màn hình nền độc lập, API `vd.xxx` song song `d.xxx`) — có thể chạy automation nền trong khi dùng màn hình chính bình thường.

## Yêu cầu triển khai
- **Trên máy Android**: cài app `firerpa.apk` (tải từ docs Quick Start), cấp quyền root (Magisk ≥ 20.4, hoặc module Magisk `lamda-magisk-module.zip` tự autostart sau reboot 30s) hoặc chạy không root qua **Shizuku** (một số tính năng đặc quyền bị giới hạn). Hỗ trợ arm64/armv7/x86/x86_64, Android 6–17 (device-farm.com ghi Android 6–17, uptime claim 99.9%).
- **Cài đặt tay (pro)**: push `lamda-server-arm64-v8a.tar.gz` qua adb, giải nén vào `/data` (root) hoặc `/data/local/tmp` (shell), chạy `sh /data/server/bin/launch.sh`. Changelog v7.25 từng fix "launch failure from Termux" → service đã từng được chạy từ Termux, nhưng docs chính thống không mô tả flow Termux chi tiết.
- **Phía PC**: Python ≥ 3.6 (changelog v5.0: client hỗ trợ Python 3.11), `pip install lamda` (PyPI có bản 9.3).
- **BẢO MẬT — quan trọng**: mặc định FIRERPA **không bật authentication** — "bất kỳ ai cũng truy cập được mọi thứ trên máy". Phải bật TLS + service certificate (APK: trong Settings; Magisk/manual: nạp `lamda.pem` theo chương Server Certificate) hoặc chỉ chạy trong mạng tin cậy. Remote desktop hỗ trợ đặt mật khẩu WebUI, API exclusive lock chống client tranh chấp.
- **MCP/Agent (điểm mạnh nhất cho use case của Mạnh)**: MCP server built-in tại path `/mcp/` (streamable-http, hỗ trợ tool calls/resource/prompts/progress/logs), tương thích Claude, Cursor và các MCP client khác; lệnh **`agent`** điều khiển máy bằng **ngôn ngữ tự nhiên qua bất kỳ OpenAI-compatible API nào có tool calls**, hỗ trợ **Vision mode** + "OpenAI semantic task executor", kết hợp crontab để chạy task AI theo lịch. → Không bắt buộc dùng API trả phí: có thể trỏ sang OpenRouter (tương thích OpenAI). Lưu ý: model phải hỗ trợ **tool calls + vision** mới lái được máy (poolside/laguna-s-2.1:free có hỗ trợ tool calls hay không — chưa xác minh được; model vision nvidia/nemotron-3.5-super-vl:free đã được Mạnh xác minh trước đó là phù hợp bước "nhìn màn hình").

## Docs có đủ để dựng không?
Đủ để dựng cơ bản và trung cấp: docs gốc tại device-farm.com/docs/en (Quick Start, Install Server — hướng dẫn APK/Shizuku/Magisk/manual từng bước, có cảnh báo bảo mật rõ ràng) và file **llms-full.txt** (~21.5k dòng, cập nhật 23/10/2026) gộp toàn bộ tài liệu + changelog chi tiết từng version. Thiếu: cơ chế nội bộ của distributed task system (chỉ ghi "built-in" từ v9.0), giá/điều khoản networking subscription service, và flow chạy service từ Termux không được tài liệu hóa (chỉ có dòng changelog v7.25).

## Verdict — Mạnh deploy được không?
**Deploy được và là ứng viên rất mạnh cho vai trò "tay điều khiển" 2 máy.** S8+ (root) → full năng lực (Frida, MITM, setprop ro.*, SELinux); S20FE chưa root → cài qua Shizuku, chấp nhận giới hạn tính năng đặc quyền. Cả hai máy đã trong Tailscale mesh nên có thể truy cập service một-cổng của lamda qua Tailscale IP (kèm TLS/certificate vì mặc định không có auth) — không cần Pigeon/Hub phức tạp ở giai đoạn đầu; remote desktop trên trình duyệt + MCP server `/mcp/` + lệnh `agent` trỏ về OpenRouter free cho phép ZeroClaw/Muse lái máy từ xa. Không thấy yêu cầu subscription trả phí bắt buộc cho chức năng cốt lõi. **Caveat trung thực**: (1) chưa deploy thực tế nên hiệu năng H.264/WebRTC trên S8+ đời cũ và độ ổn định 24/7 cần test tay; (2) Pigeon (đổi tên từ hub) còn quá non (46 stars/8 commit) — đừng dựng production trên nó ngay; (3) xác minh model free OpenRouter có tool calls trước khi dùng lệnh `agent`; (4) auto thao tác hàng loạt trên Facebook/TikTok qua tool này vẫn chịu rủi ro khóa nick như vụ 06/10 — chỉ nên dùng cho thao tác nhẹ/đọc và test.

## Nguồn
- https://github.com/firerpa/lamda — README (mô tả tổng, kiến trúc C/S, multi-device, MCP/agent, 160+ APIs, triển khai APK/Magisk/manual, yêu cầu bảo mật)
- https://device-farm.com/docs/en/ — Quick Start (cài firerpa.apk root/Shizuku, bật service, remote desktop qua Console URL)
- https://device-farm.com/docs/en/install-server — Install Firerpa Server (Magisk module `lamda-magisk-module.zip`, cài tay `lamda-server-arm64-v8a.tar.gz`, mặc định không authentication, nạp `lamda.pem`, preinstall `adb_keys`)
- https://device-farm.com/llms-full.txt — toàn bộ docs (chương Distributed & Multi-Device Management: Pigeon + hub-bridge, SAPI, P2P Bridge, distributed task system từ v9.0, frp/OpenVPN self-host, mDNS; chương AI/MCP/Agent: `/mcp/` streamable-http, lệnh `agent` qua OpenAI-compatible API + vision + crontab; changelog v7.25 "Fixed launch failure from Termux", v7.50 networking subscription, v9.0 P2P Bridge)
- https://github.com/firerpa/pigeon — Pigeon platform (repo hub đổi tên; Docker deploy, 46 stars, 8 commits, admin/pigeon mặc định)
- https://device-farm.com/ — trang chủ (claim Android 6–17, Python API mẫu `from lamda.client import *`)
