# SwarmForge

- URL: https://github.com/GavinGeizer/swarmforge-oss
- Stars: 2 (tại thời điểm đọc)
- Đánh giá ngày: 23/10/2026

## Là gì (2 dòng)

Orchestrator tự host cho các coding agent chạy song song: AI lead giao task qua MCP, coordinator tự động provision **mỗi worker một VM riêng trên nền tảng Freestyle**, chạy OpenCode bên trong, rồi thu artifact + push Git branch có verify. Trạng thái durable trong SQLite, có dashboard terminal.

## Kiến trúc multi-device (chi tiết)

- **Coordinator** chạy trên **Linux x64 glibc duy nhất** (binary phát hành hoặc build từ source bằng Bun ≥1.4.2; chưa có bản macOS/Windows/ARM64/Alpine). Một coordinator ↔ một SQLite DB trên storage persistent. Nguồn: https://github.com/GavinGeizer/swarmforge-oss
- **Worker = 1 VM Freestyle riêng biệt**, provision từ snapshot do người dùng chuẩn bị trước. Snapshot bắt buộc có: OpenCode tương thích SDK 1.18.31, Python 3, Git, Bash, systemd. Luồng: lead gọi `spawn_worker` qua MCP (Streamable HTTP, `http://127.0.0.1:8787/mcp`) → coordinator queue → gọi Freestyle API tạo VM → clone repo → chạy OpenCode session. Nguồn: https://github.com/GavinGeizer/swarmforge-oss
- **Giao việc bằng MCP** (Model Context Protocol): lead là bất kỳ MCP client nào (ví dụ OpenCode config mẫu trong README). Tool: `spawn_worker`, follow-up message, pause/resume/cancel, đọc structured result, thu artifact, destroy VM. Worker không tự trao đổi/merge với nhau; lead tự chia task qua Git + follow-up. Nguồn: https://github.com/GavinGeizer/swarmforge-oss
- **Model:** endpoint **OpenAI-compatible chat-completions có tool-calling**, 1 endpoint/model cho cả deployment, do người dùng cung cấp (`SWARMFORGE_MODEL_BASE_URL`, `..._API_KEY`, `..._MODEL_NAME`). OpenRouter (`https://openrouter.ai/api/v1`) đúng chuẩn này → **dùng được với model free**, miễn endpoint reach được từ VM. Không bundle inference sẵn. Nguồn: https://github.com/GavinGeizer/swarmforge-oss
- **Handoff:** worker push branch qua GitHub App / OAuth / SSH, coordinator verify remote commit (SHA-256 cho artifact). Không tự mở PR/merge. Nguồn: https://github.com/GavinGeizer/swarmforge-oss
- **Điểm mấu chốt:** `WorkerProvider`/`CodingAgent` chỉ là extension point trong code — **bản phát hành chỉ có `FreestyleProvider` + `OpenCodeAgent`**. Không có adapter local-process, Docker, Kubernetes, SSH-host hay VM provider khác. Thiết bị của Mạnh (A1, điện thoại) **không thể làm worker**. Nguồn: https://github.com/GavinGeizer/swarmforge-oss

## Yêu cầu triển khai

- Tài khoản **Freestyle** + `FREESTYLE_API_TOKEN` + `FREESTYLE_SNAPSHOT_ID` (tự build snapshot đủ điều kiện OpenCode). Pricing (freestyle.sh/pricing): trả theo usage — $0.04032/vCPU-giờ, $0.0129/GiB-giờ; gói **Free gồm 200 vCPU-giờ + 400 GiB-giờ/tháng**, hết là hard-stop (không trừ tiền thêm); giới hạn 10 VM đồng thời. Ví dụ worker 2vCPU/4GiB ≈ $0.13/giờ → free tier đủ ~100 giờ worker 2vCPU/tháng. **Mỗi worker spawn là một VM billable** (kể cả VM retained sau khi xong việc nếu không destroy). Nguồn: https://github.com/GavinGeizer/swarmforge-oss , https://www.freestyle.sh/pricing
- Coordinator đặt được trên A1 Flex (Linux x64) — nhưng worker vẫn chạy trên cloud Freestyle (Mỹ), không phải hạ tầng của Mạnh.
- License: **PolyForm Small Business 1.0.0** — source-available, hạn chế dùng cho mục đích thương mại (không phải open-source OSI). Dùng cá nhân/nghiên cứu của Mạnh thì trong phạm vi cho phép, nhưng cần đọc kỹ trước khi dùng cho việc có doanh thu. Nguồn: https://github.com/GavinGeizer/swarmforge-oss

## Docs có đủ để dựng không?

Đủ và chi tiết bất thường cho repo 2 stars: README ~900 dòng (before-you-start → quickstart `init`/`doctor`/`serve` → kết nối MCP → chạy task đầu tiên → thu artifact → dọn VM → troubleshooting), `docs/` có implementation architecture, MCP tool reference, worker result protocol, worker snapshot checklist, site getswarmforge.tech có setup guide, 2 releases (đang v0.1.3), CI verify trên Linux x64. Tác giả có bài dev.to giới thiệu. Nguồn: https://github.com/GavinGeizer/swarmforge-oss , https://dev.to/gavingeizer/i-built-an-open-source-orchestrator-for-ai-coding-agents-5g5e

## Verdict — Mạnh deploy được không?

**Loại.** Lý do: (1) worker **bắt buộc là VM Freestyle trên cloud** (trả theo usage, free tier giới hạn ~100 giờ worker 2vCPU/tháng và hết là dừng) — không tận dụng được A1 Flex/điện thoại/Botkeep của Mạnh, đi ngược mục tiêu chi phí ~$0 và triết lý tự host; (2) coordinator chỉ Linux x64, hệ sinh thái khép kín quanh Freestyle + OpenCode; (3) repo 2 stars, một tác giả, v0.1.3 rất non. Điểm cộng duy nhất: OpenRouter free gắn được vào làm model endpoint. Nếu sau này Mạnh chấp nhận trả tiền VM cloud cho worker thì xem lại, hiện tại không.

## Nguồn (URL cụ thể đã đọc)

- https://github.com/GavinGeizer/swarmforge-oss (README đầy đủ: kiến trúc, supported harnesses/models/infrastructure, quickstart, license)
- https://www.freestyle.sh/pricing (giá VM, free allowance, hard-stop)
- https://dev.to/gavingeizer/i-built-an-open-source-orchestrator-for-ai-coding-agents-5g5e (bài giới thiệu của tác giả)
