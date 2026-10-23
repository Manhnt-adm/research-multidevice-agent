# yomo
- URL: https://github.com/yomorun/yomo
- Stars: 1929 (tại thời điểm đọc 23/10/2026)
- Đánh giá ngày: 23/10/2026
## Là gì (2 dòng)
Framework serverless cho **LLM Function Calling** (AI agent tools): viết handler bằng TypeScript/Go, triển khai như "serverless functions" và cho LLM gọi qua API tương thích OpenAI `/v1/chat/completions` hoặc MCP server. Mã nguồn chính viết bằng **Rust** (CLI + Zipper server).
## Kiến trúc multi-device (chi tiết: edge node giao tiếp ra sao, có khái niệm fleet/worker đa máy không)
- Mô hình 3 thành phần (theo glossary chính thức https://www.yomo.run/docs/glossary):
  - **Source** — sinh dữ liệu, gửi về Zipper.
  - **Zipper** — node trung chuyển: forward dữ liệu giữa Source và StreamFunction; Zipper là **QUIC server** (TLS v1.3), Source/StreamFunction là QUIC client.
  - **StreamFunction** — hàm xử lý dữ liệu, có thể build ra `.wasm` (v1) hoặc chạy như tool TS/Go kết nối tới Zipper (v2).
- **"Geo-distributed" = mesh Zipper**: "Zipper Cascading and Mesh Network — Zippers can be connected to each other... the data will be delivered to the nearest StreamFunction" (https://www.yomo.run/docs/glossary). Mục đích là giảm latency inference khi đặt tool gần user, **không phải điều phối agent**.
- **KHÔNG có khái niệm fleet/worker đa máy kiểu agent-orchestrator**: không có dispatcher, claim task, conductor, gating hay supervision. `yomo run -n <tool>` chỉ kết nối một tool tới một Zipper endpoint (`YOMO_SFN_ZIPPER`, ví dụ `localhost:9000` hoặc `zipper.vivgrid.com:9000`). File `src/agent_loop.rs` trong repo là vòng lặp LLM↔tool-call, không phải vòng lặp điều phối agent.
- "Edge phân tán địa lý" ở đây thực chất là **nền tảng function-as-a-service cho tool của LLM** (cạnh tranh với việc tự host tool API / MCP server), không phải hệ thống chạy đội agent trên nhiều máy.
## Yêu cầu triển khai (cần cài gì, phụ thuộc nặng nhẹ, phần cứng yêu cầu)
- **CLI**: `curl -fsSL https://get.yomo.run | sh` — cài binary Rust `yomo` (nhẹ).
- **Server**: `yomo serve` (chạy Zipper + HTTP API trên port 9000/9001, config `yomo.yml`).
- **LLM provider**: quickstart chính thức dùng **Ollama** (`ollama pull ornith` — model local, nặng, không hợp máy yếu). Tuy nhiên config `yomo.yml` hỗ trợ `type: "openai-compatible"` với `base_url` tùy ý (ví dụ OpenRouter) — không bắt buộc Ollama (https://github.com/yomorun/llm-function-calling-examples, phần "YoMo Usage").
- **Runtime cho tool**: Node.js (TS) hoặc Go binary cho từng function; mỗi tool là một process riêng kết nối QUIC tới Zipper.
- Có hướng deploy managed trả phí qua **Vivgrid** (`yc deploy`, credential `vg-<token>`) — trùng với ràng buộc "không subscription trả phí" của Mạnh.
## Docs có đủ để dựng không? (đánh giá trung thực)
- **Đủ cho mục đích của nó** (host tool LLM): README có quickstart đầy đủ 3 bước (install → serve → init/run tool → gọi API), docs CLI tại https://yomo.run/docs/cli, config mẫu `yomo.yml` trong repo llm-function-calling-examples. Chưa xác minh được docs về Zipper cascading/mesh sâu cỡ nào — glossary chỉ mô tả ở mức khái niệm.
- Nhưng docs **không giúp gì** cho use case "điều phối agent đa máy" vì đó không phải thứ YoMo làm.
## Verdict — Mạnh deploy được không? (chọn 1: Được / Được có điều kiện / Loại — kèm 1–3 dòng lý do gắn với hạ tầng của Mạnh)
**Loại** — YoMo là nền tảng host "tool/function cho LLM gọi" (function-as-a-service cho AI agent), không phải orchestrator điều phối agent trên nhiều máy: không có dispatcher, task claim, hay fleet management. Mạnh có thể tự host `yomo serve` trên A1 Flex và cho ZeroClaw gọi tool qua OpenAI-compatible API, nhưng đó là việc triển khai **infrastructure cho tool**, không giải bài toán "1 conductor + agents trên S8+/S20FE/VPS". Dùng sai công cụ cho bài toán.
## Nguồn (liệt kê URL cụ thể đã đọc)
- https://github.com/yomorun/yomo (README, root/src listing, stars 1929)
- https://www.yomo.run/ (Introduction: "Open source Serverless LLM Function Calling framework for AI agents")
- https://www.yomo.run/docs (Getting Started — Streaming Serverless Framework)
- https://www.yomo.run/docs/cli (lệnh `yomo serve` / `run` / `init` / `dev`)
- https://www.yomo.run/docs/glossary (kiến trúc Source–Zipper–StreamFunction, Zipper Cascading and Mesh Network)
- https://github.com/yomorun/llm-function-calling-examples (config `yomo.yml` với openai-compatible provider, env `YOMO_SFN_ZIPPER`, deploy Vivgrid)
- https://pkg.go.dev/github.com/yomorun/yomo@v1.16.0 (mô tả v1: Streaming Serverless Framework trên QUIC)
