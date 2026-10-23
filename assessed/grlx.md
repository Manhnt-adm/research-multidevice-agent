# grlx

- URL: https://github.com/gogrlx/grlx
- Stars: 226 (tại thời điểm đọc)
- Đánh giá ngày: 23/10/2026

## Là gì (2 dòng)
Công cụ config management fleet viết thuần Go (binary nhỏ, ít tốn tài nguyên), kiến trúc 3 thành phần: `farmer` (server điều khiển), `sprout` (agent chạy trên máy cần quản lý), CLI `grlx`. Recipe là file YAML khai báo "desired state", `cook` để áp lên nhiều máy song song.

## Pattern đáng học cho Mạnh (2–4 dòng cụ thể)
1. **NATS embedded trong farmer:** bus message Pub-Sub có sẵn trong server điều khiển, node subscribe nhận lệnh thay vì phải cài broker riêng — nhẹ, hợp triết lý ~0 của Mạnh. NATS cũng hỗ trợ request/reply → có thể làm heartbeat + task dispatch ngay trên bus, không cần HTTP polling.
2. **Xác thực public-key + TLS mặc định:** sprout tự xác thực bằng key pair, CLI duyệt key mới (`grlx keys accept`) — mô hình enrollment 2 lớp đúng cái mà MVP anti-fake của Mạnh còn thiếu (device enrollment).
3. **Target theo glob (`grlx -T \* cmd run ...`):** gửi lệnh 1 lần chạy song song cả fleet — đúng nhu cầu "ra lệnh cho nhiều máy" của Mạnh.

## Verdict
**Dùng được pattern** — NATS embedded + key enrollment + target glob là bộ sườn điều khiển fleet đáng cắp nhất cho agent hub của Mạnh (đặt trên A1 Flex chạy sau Tailscale). Repo nhỏ (226★) nhưng đang active; chỉ cắp kiến trúc, không cần dùng Go/YAML recipe nếu Python stack đã ổn.

## Nguồn (URL cụ thể đã đọc)
- https://github.com/gogrlx/grlx (README: Quick Start, Architecture, Batteries Included)
