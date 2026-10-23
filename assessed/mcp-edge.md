# MCP-Edge
- URL: https://github.com/jemsbhai/mcp-edge
- Stars: 3 (qua GitHub API, tại thời điểm đọc 23/10/2026; 1 fork, 1 issue mở, push mới nhất **08/07/2026** — repo đã ~3,5 tháng không commit, có dấu hiệu stale)
- Đánh giá ngày: 23/10/2026

## Là gì (2 dòng)
MCP-Edge (Python, MIT) **mở rộng giao thức MCP xuống thiết bị edge/IoT**: một gateway chạy trên máy Linux-class nối MCP client (SSE/HTTP/stdio) tới thiết bị phần cứng qua serial (UART), BLE, Wi-Fi (TCP), biến mỗi thiết bị thành một MCP tool provider; kèm **MCP-Lite** — MCP server siêu nhẹ cho MCU chỉ ~512KB RAM.

## Kiến trúc multi-device (chi tiết)
- **Gateway** (`src/mcp_edge/gateway.py`): chạy trên máy Linux-class (aarch64). Nhận call từ MCP client, định tuyến tới thiết bị trong `DeviceRegistry` theo namespace `device-id/tool` (ví dụ `sensor-01/read_temp`), chuyển qua transport vật lý (serial/BLE/TCP), encode **CBOR** qua wire format length-prefixed. Có schema caching, connection pooling, offline request buffering, health monitor.
- **MCP-Lite** (`src/mcp_edge/mcplite.py`): server MCP rút gọn chạy trên MCU — ví dụ firmware ESP32 MicroPython trong `examples/firmware/esp32_micropython/`, đã boot được trên Wokwi (mô phỏng) cho ESP32/S3/C3/C6.
- **Four-tier device taxonomy** (`src/mcp_edge/tiers.py`): phân loại MCU bị giới hạn → smart IoT node → BLE-only wearable → máy Linux-class, mỗi tier có MCP strategy riêng.
- **Docs** `docs/platform-support.md` ghi rõ ràng: gateway đã verify cài + boot trên aarch64 Linux và Raspberry Pi OS (QEMU) trong CI; ESP32 chỉ là **Wokwi boot smoke test**; RP2040/nRF52840 là stretch goal; AVR (UNO R3/Mega, RAM vài KB) không host được MCP-Lite.
- ⚠️ **Điểm quan trọng về câu hỏi kiểm chứng** ("agent trên hub gọi tool trên thiết bị edge được không"): CÓ — nhưng hướng dùng là **agent LLM ở cloud gọi tool của phần cứng vật lý** (đọc cảm biến, đọc nhịp tim…), không phải agent hub điều khiển thực thi lệnh trên một máy/phần mềm edge. Điện thoại S8+/S20FE của Mạnh là máy Linux-class (Android/Termux) — MCP-Edge không cho "chạy lệnh trên phone", nó cho "phone/server gọi tool của MCU". Abstraction sai với nhu cầu điều khiển fleet của Mạnh.

## Yêu cầu triển khai
- Gateway: Python **3.10+**, `pip install mcp-edge` (+ extras: `serial` cho pyserial, BLE qua bleak, Wi-Fi discovery qua zeroconf). Tài nguyên: chạy được trên Pi 4/5, máy Linux-class nhẹ — chưa có con số RAM cụ thể trong docs.
- Device side: MCU chạy được MicroPython (ESP32 family có firmware mẫu; UNO R4/Giga "needs a firmware port" — chưa có).
- ⚠️ README ghi rõ đường serial là **experimental** và "**not yet validated on physical hardware**" — transport vật lý mới test bằng phần mềm/Wokwi.

## Docs có đủ để dựng không?
Vừa đủ cho demo: README có quickstart (`mcp-edge run --demo`, ví dụ gateway-as-library đầy đủ, ví dụ SerialTransport), `docs/platform-support.md` trung thực về mức độ verify (sim vs hardware). Đủ để dựng demo in-process và thử gateway trên Pi; **không đủ tự tin** để dựng đường serial/BLE/Wi-Fi vật lý thật (tự nhận experimental, chưa validate trên hardware thật).

## Verdict — Mạnh deploy được không?
**Loại.** (1) Abstraction sai: đây là gateway IoT (agent cloud → tool của MCU/sensor), không phải công cụ điều khiển fleet điện thoại/server của Mạnh — rook (đợt 2) hoặc các repo device-control đã phù hợp hơn; (2) repo **stale** (~3,5 tháng không commit, 3 stars, 1 maintainer); (3) trạng thái alpha, API unstable, transport vật lý chưa validate trên hardware thật. Chỉ nên xem lại nếu một ngày Mạnh muốn gắn ESP32/cảm biến vật lý vào hệ agent của mình.

## Nguồn (URL cụ thể đã đọc)
- https://github.com/jemsbhai/mcp-edge (README.md: gateway + MCP-Lite + four-tier taxonomy, status alpha, quickstart demo `mcp-edge run --demo`, gateway-as-library, SerialTransport experimental "not yet validated on physical hardware", roadmap v0.1/v0.2)
- https://github.com/jemsbhai/mcp-edge/blob/main/docs/platform-support.md (install path gateway vs device firmware, verify bằng CI: aarch64 native + Raspberry Pi OS QEMU; ESP32 Wokwi boot smoke test; UNO R4/Giga needs firmware port; AVR cannot host)
- https://github.com/jemsbhai/mcp-edge/tree/main/src/mcp_edge (xác minh code: gateway.py, mcplite.py, registry.py, tiers.py, transports/serial|ble|tcp|udp, client.py, server.py stdio)
- https://api.github.com/repos/jemsbhai/mcp-edge (stars 3, fork 1, open issues 1, pushed_at 2026-07-08, created 2026-06-25)
