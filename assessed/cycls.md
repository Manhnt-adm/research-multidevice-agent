# cycls
- URL: https://github.com/Cycls/cycls
- Stars: 27 (tại thời điểm đọc)
- Đánh giá ngày: 23/10/2026

## Là gì (2 dòng)
SDK Python để xây và deploy AI agent lên **Cycls Cloud** của chính họ (SaaS). Câu "distributing AI agents" trong tagline là marketing cho việc deploy containerized agent lên cloud của Cycls, không phải điều phối agent trên dàn máy của người dùng.

## Kiến trúc multi-device (chi tiết vừa đủ)
- 4 primitive khai báo (Image, Web, LLM, Volume) + 3 decorator (`@cycls.function`, `@cycls.app`, `@cycls.agent`) — tất cả đóng gói thành **Docker container**.
- Chạy local: `cycls run my_agent.py` (Docker + hot-reload, localhost:8080). Deploy: `cycls deploy my_agent.py` → live tại `https://my-agent.cycls.ai`, cần API key từ cloud.cycls.com.
- Chạy `--remote` là cloud dev loop của Cycls, không cần Docker local.
- Không có khái niệm cluster/fleet/mesh: agent chạy như một service ASGI trong container trên hạ tầng Cycls; multi-provider LLM chỉ là routing model (Anthropic native + mọi endpoint OpenAI-compatible qua `base_url`), không phải phân tán compute.

## Yêu cầu triển khai
- `pip install cycls`; Docker nếu muốn chạy local.
- Tài khoản + API key tại cloud.cycls.com (đăng ký ngoài, chưa xác minh gói free/paid).
- Không có bằng chứng có thể tự host control plane trên máy của mình — deploy target duy nhất được nêu là Cycls Cloud.

## Docs có đủ để dựng không?
- README đầy đủ cho flow SDK → deploy lên Cycls Cloud; có docs.cycls.com được link (chưa đọc sâu do giới hạn rate).
- **Không đủ** cho nhu cầu self-host multi-device của Mạnh: docs đã đọc không mô tả cách dựng Cycls trên hạ tầng riêng.

## Verdict — Mạnh deploy được không?
**Loại.** "Distributing AI agents" ở đây nghĩa là "deploy agent lên cloud của Cycls", không phải chạy agent trên nhiều máy vật lý của Mạnh (VPS/điện thoại). Nó là SDK build agent + PaaS, không phải orchestrator; không giải quyết bài toán fleet multi-device. (27 sao, 0 release, repo không có dấu hiệu self-host.)

## Nguồn (URL cụ thể đã đọc)
- https://github.com/Cycls/cycls (README: primitives, Running, deploy → my-agent.cycls.ai, API key tại cloud.cycls.com)
- https://pypi.org/project/cycls (xác nhận cùng nội dung README: deploy production → https://my-agent.cycls.ai)
