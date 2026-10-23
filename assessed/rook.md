# rook
- URL: https://github.com/Bake-Ware/rook
- Stars: 4 (rất mới, ít người dùng)
- Đánh giá ngày: 23/10/2026

## Là gì (2 dòng)
Rook là nền tảng quản lý fleet tự host: cài một **worker** nhỏ trên mỗi máy (server, RPi, laptop, VPS, điện thoại Android, dongle USB), worker tự dial-out về **hub** do mình host; điều khiển tất cả từ một dashboard web, terminal UI, hoặc AI agent qua MCP. Toàn bộ kênh mã hóa end-to-end (ChaCha20-Poly1305 trên PSK), relay chỉ chuyển tiếp "mù" không cầm key.

## Kiến trúc multi-device
- **Hub ↔ worker**: hub-and-spoke qua một **relay mù** (`telesthete-hub`, viết bằng Rust). Worker **dial-out về hub** (không cần mở port trên máy worker — cực hợp NAT/router nhà).
  - LAN: worker nói UDP trực tiếp tới relay (port 7474/udp).
  - Remote: worker nối WebSocket tới endpoint `/band` của hub (qua HTTPS reverse proxy/tunnel trên port 443).
- **Mã hóa end-to-end**: mỗi "band" là một nhóm peer chia sẻ một **pre-shared key (PSK)** — khuyến nghị dạng 5 từ tiếng Anh đọc được (~64.6 bit entropy), hoặc tạo PSK 192-bit bằng `python -m rook.remote.psk`. Key mã hóa AEAD sinh ra từ PSK (ChaCha20-Poly1305); `band_id = SHA256(PSK)[:16]` là nhãn định tuyến duy nhất ở dạng cleartext. **Relay không cầm key nên không đọc/giả mạo được traffic** — nó chỉ forward theo `band_id`.
- **Worker join**: có 2 cách. (1) **Pairing code**: trên dashboard bấm "Get a pairing code" (code 6 ký tự, hết hạn sau 5 phút), chạy lệnh cài one-line trên máy mới (`curl -fsSL 'https://<hub>/worker?band=CODE' | bash`; có biến thể PowerShell và Android/Termux). (2) Chạy tay: `rook worker --hub <hub>:7474 --psk … --name …` (hoặc `--hub domain:443 --ws` cho remote). Worker announce bản thân mỗi 30s (tên, version, capabilities); mất tín hiệu ~90s thì bị gỡ khỏi roster.
- **Trust bổ sung**: OTA update và lệnh deauth được **ký riêng bằng ed25519 key** của hub (band membership một mình không đủ để push code lên worker). Hub cũng có policy phân quyền (audit/enforce) và approval gate cho lệnh nguy hiểm của agent.
- **Gap bảo mật được thừa nhận**: chưa có per-worker identity trên band (peer độc hại giữ PSK không thể cắt riêng mà phải rotate cả key); deauth chỉ tác dụng với worker hợp tác; chính chủ ghi rõ "early, single-maintainer software" và "ai nắm band key thì chạy lệnh được trên mọi worker — coi key như root password".

## Yêu cầu triển khai
- **Hub** (chạy trên một máy Linux): 3 cách — (a) one-command installer (systemd, cần `curl`+`sudo`), (b) Docker image `ghcr.io/bake-ware/rook` (amd64 **và arm64**), (c) từ source (Python **3.11+**, venv; relay có prebuilt x86_64/aarch64/armv7, nếu không có thì cần Rust/cargo để build). Hub = 2 tiến trình nhỏ: relay UDP 7474 + `rook-hub` (dashboard + MCP server + /band bridge) trên TCP 7005. State gói gọn trong một thư mục `ROOK_DATA_DIR`.
- **Worker**: **pure-Python**, plugin-based, chạy `rook worker` hoặc file bundle `band-worker.pyz` — nhẹ, không đòi Rust/C toolchain trên máy worker. Installer một dòng cho Linux/macOS/Windows.
- **Android**: có 2 đường:
  - **App native (APK, Kotlin + Chaquopy)**: chạy worker Python như foreground service (chống Doze tốt hơn Termux), screenshot màn hình thật qua MediaProjection, tap/swipe/type qua AccessibilityService **không cần root**, thêm SMS/contacts/location/battery/torch/clipboard/unlock… Hub serve APK tại `/apk`, tự update (check chữ ký SHA-256 + cert).
  - **Termux**: installer one-line cho Android/Termux trên trang cài đặt; shell đầy đủ nhưng screenshot chỉ chụp camera, `hid.*` cần root.
- **Phần cứng**: không yêu cầu đặc biệt — hub chạy được trên Pi/VPS; worker Python nhẹ (thực tế sẽ rất nhẹ, nhưng chưa thấy con số RAM cụ thể nào trong docs — chưa xác minh được mức MB chính xác).

## Docs có đủ để dựng không?
Rất đầy đủ cho một repo 4 sao: README chi tiết (quickstart 3 cách cài hub, kiến trúc, security model), `docs/FEATURES.md` (tour ~1000 dòng: band/pairing, capabilities, hardware integration, installers), `docs/operations/install.md` (từng bước cài hub, troubleshooting, nâng cấp/gỡ), `docs/operations/settings-reference.md` (mọi env var), `android/` có README riêng đầy đủ (build APK, cấu hình, capability parity), `docs/spec/` + `conformance/` cho giao thức telesthete. Thiếu duy nhất con số benchmark tài nguyên worker (RAM/CPU thực tế).

## Verdict — Mạnh deploy được không?
**Được — đây là repo phù hợp nhất trong track cho use case của Mạnh.** Worker pure-Python nhẹ chạy được trên S8+ qua Termux (hoặc build APK native để có screenshot/tap không root, đúng thứ skill phone-control của anh cần), hub đặt trên VPS Oracle hoặc A1 Flex sau này bằng one-command installer/Docker; worker dial-out nên Tailscale chỉ là option, không bắt buộc; không cần subscription AI (MCP server tự host, tự cắm model free qua OpenRouter được). Điểm trừ: repo mới (tháng 3/2026), 4 sao, một maintainer, tự nhận "early" — có rủi ro đổi interface/API và bug bảo mật chưa được harden; PSK là single point of failure cho cả band (mất key = mất cả fleet). Nên dựng thử trên VPS + 1 điện thoại test trước, rotate PSK ngay nếu key từng bị dán ra chat/log.

## Nguồn
- https://github.com/Bake-Ware/rook (README: tổng quan, quickstart, kiến trúc, security model, repository layout)
- https://github.com/Bake-Ware/rook/blob/master/docs/FEATURES.md (feature tour: band/pairing/PSK, capabilities, hardware integrations, Android app)
- https://github.com/Bake-Ware/rook/tree/master/android (native Android worker app: build, capability parity, Termux so sánh)
- https://github.com/Bake-Ware/rook/blob/master/docs/operations/install.md (cài hub 3 cách, yêu cầu, troubleshooting, nâng cấp)
- https://github.com/Bake-Ware/rook/tree/master/docs (mục lục docs: design/, operations/, spec/)
