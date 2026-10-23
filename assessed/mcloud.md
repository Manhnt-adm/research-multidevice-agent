# mcloud (Zebrunner)
- URL: https://github.com/zebrunner/mcloud
- Stars: 204 (tại thời điểm đọc)
- Đánh giá ngày: 23/10/2026

## Là gì (2 dòng)
Device farm cho Android/iOS: hạ tầng tập trung quản lý nhiều điện thoại/máy tính bảng/TV vật lý, phục vụ chạy kiểm thử tự động (Appium) và truy cập màn hình thiết bị từ xa qua web cho người vận hành. Là fork của OpenSTF/DeviceFarmer-stf thêm hỗ trợ iOS và đăng nhập Zebrunner; code nguồn đã khá lâu đời (repo tạo 2019-07-26).

## Kiến trúc multi-device (hub quản lý farm điện thoại như thế nào)
- **Hub (server)**: `docker-compose` nhiều service — nginx (`stf-proxy`) làm entry point; các unit STF (app/api/auth/websocket/processor/reaper/triproxy/storage); RethinkDB làm database; `mcloud-grid` là Selenium/Appium hub gắn trước STF; quản lý bằng script `zebrunner.sh` (setup/start/stop/restart). URL: `http://hostname:80/stf` (UI), `/mcloud/wd/hub` (Appium hub), `/mcloud/grid/console` (console thiết bị). Đăng nhập mặc định `auth-mock` (tên/email bất kỳ), token truy cập tạo trong Settings → Keys.
- **Agent trên máy cắm thiết bị**: repo riêng `mcloud-agent` (https://github.com/zebrunner/mcloud-agent) — chạy trên máy Linux/macOS mà **điện thoại cắm vật lý qua cổng USB**. Triển khai bằng Ansible playbook: whitelist thiết bị theo udid, tạo udev rules, và **mỗi thiết bị sinh ra một bộ container Docker riêng**: `device-<Name>-<udid>-connector` (adb/usbmuxd), `-appium`, `-stf`, `-uploader`. Điện thoại phải bật USB Debugging, cắm trực tiếp USB (không phải qua mạng như use case của Mạnh).
- **Remote control**: qua giao diện web STF (stream màn hình + điều khiển chuột/phím), và automation qua Appium protocol tới hub.

## Yêu cầu triển khai (cần cài gì, nặng nhẹ, có cần hạ tầng đặc biệt không)
- **Hub server**: Ubuntu 24.04 / CentOS 7+ / Amazon Linux 2 / macOS; CPU 4+ cores, **RAM 16 GB**, SSD 32 GB+, Docker v27+ compose, git — theo bảng "System requirements" README gốc.
- **Máy agent (máy cắm điện thoại qua USB)**: yêu cầu CÒN CAO HƠN hub — CPU 8+ cores, **RAM 32 GB**, SSD 128 GB+, Docker, Ansible 2.10+, trên Linux (Android) hoặc macOS (iOS cần thêm go-ios, socat, jq, Xcode + Apple Developer account để build WebDriverAgent.ipa).
- Hạ tầng đặc biệt: cần máy vật lý/VM đủ mạnh để **cắm USB vật lý** các điện thoại; container chạy per-device (mỗi máy ~4 container + appium). iOS bắt buộc macOS + tài khoản Apple Developer — không liên quan nhưng cho thấy độ nặng của stack.

## Docs có đủ để dựng không?
Đủ và chi tiết: README mcloud (setup 9 bước, bảng biến `.env`/`variables.env`, lệnh `zebrunner.sh`, endpoints, upgrade) và README mcloud-agent (hardware requirements, ansible playbook, udev/launchd, troubleshooting container per-device, build WebDriverAgent.ipa). CI chạy lint + docker compose test. Điểm trừ: là hệ sinh thái nhiều repo (mcloud, mcloud-agent, mcloud-device, mcloud-grid, appium, stf fork) — phải đọc chéo nhiều repo mới dựng trọn.

## Verdict — Mạnh deploy được không cho 2 điện thoại?
**Loại.**
- Lý do 1 — Sai mô hình: mcloud yêu cầu điện thoại **cắm USB vật lý vào một máy host mạnh** (agent host 8 cores/32 GB RAM). 2 điện thoại của Mạnh (S8+/S20FE chạy Termux, nối Tailscale) không có máy host nào để cắm — và hạ tầng mạnh nhất của anh (A1 Flex 12 GB sắp dựng) không đáp ứng nổi yêu cầu 16 GB của hub chứ chưa nói 32 GB của agent host.
- Lý do 2 — Sai mục đích: mcloud là farm phục vụ **kiểm thử Appium + remote web thủ công**, không phải "tay điều khiển" cho agent AI. Không có API lái máy bằng ngôn ngữ tự nhiên như lamda, cũng không có APK chạy trực tiếp trên điện thoại không root như rook.
- So sánh ngắn: **rook** thắng ở chỗ cài APK trực tiếp lên điện thoại (no-root, không cần host); **lamda** thắng ở chỗ điều khiển bằng ngôn ngữ tự nhiên qua API OpenAI-compatible; **mcloud** thua cả hai ở chi phí hạ tầng (vài chục GB RAM, máy cắm USB, Docker per-device) và không giúp gì cho kiến trúc 3 tầng của Mạnh (ZeroClaw trên Botkeep ra lệnh → S8+ làm tay qua ADB mạng).

## Nguồn (URL cụ thể đã đọc)
- https://github.com/zebrunner/mcloud (README: mô tả, System requirements 4 cores/16 GB RAM, Run 9 bước, Services, Endpoints, Components)
- https://github.com/zebrunner/mcloud-agent (README: Hardware requirements 8+ cores/32 GB RAM/128 GB SSD, Software prerequisites, Ansible playbook, Usage Android/iOS, Troubleshooting container per-device `device-<Name>-<udid>-{connector,appium,stf,uploader}`, build WebDriverAgent.ipa)
