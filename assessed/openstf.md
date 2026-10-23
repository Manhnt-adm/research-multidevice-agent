# OpenSTF (stf)

- URL: https://github.com/openstf/stf
- Stars: 13,961 (tại thời điểm đọc)
- Đánh giá ngày: 23/10/2026

## Là gì (2 dòng)
Web app điều khiển và quản lý farm thiết bị Android thật từ browser: xem màn hình realtime, chạm/gõ từ xa, cài APK, chạy shell, `adb connect` từ xa. Được CyberAgent phát triển ban đầu để quản lý hơn 160 máy.

## Pattern đáng học cho Mạnh (2–4 dòng cụ thể)
1. **Provider/worker model:** một máy "provider" cắm USB nhiều điện thoại, cài daemon gửi sự kiện trạng thái (online/offline/battery) về server trung tâm qua ZeroMQ; client (browser/conductor) subscribe sự kiện thay vì poll — đúng kiến trúc Mạnh đang cần cho 2 máy (S8+ có thể là provider, hub chỉ cần nhận event).
2. **minicap/minitouch (native C qua ADB):** pattern stream màn hình và inject touch bằng binary native thay vì protocol rườm rà — đáng tham khảo khi skill phone-control của Mạnh làm bước screencap→tap.

## Verdict
**Dùng được pattern** — nhưng project đã chết (không còn active development, bản cuối 3.4.2; development chuyển sang org DeviceFarmer, xem DeviceFarmer fork nếu cần code). Node.js 8-era + RethinkDB nặng hơn mục tiêu ~0, chỉ nên cắp kiến trúc provider/event và ý tưởng binary native, không bê nguyên stack.

## Nguồn (URL cụ thể đã đọc)
- https://github.com/openstf/stf (README, features, requirements, warning banner)
