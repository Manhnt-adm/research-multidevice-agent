# mcp-mesh
- URL: https://github.com/dhyansraj/mcp-mesh
- Stars: 42 (tại thời điểm đọc)
- Đánh giá ngày: 23/10/2026

## Là gì (2 dòng)
Framework distributed AI agent (Python/Java/TypeScript, license MIT) theo triết lý "bạn viết logic agent, mesh lo discovery/kết nối/heal/trace": agent giao tiếp qua MCP + A2A + REST. Điểm khác biệt là **Distributed Dynamic Dependency Injection (DDDI)** — agent khai báo dependency bằng decorator `@mesh.tool(capability=...)`, mesh tự discovery/inject dependency ở runtime, hot-swap không cần restart.

## Kiến trúc multi-device (ngắn gọn)
- Registry chỉ làm facilitator; agent nói chuyện trực tiếp với nhau, có self-healing (auto-reconnect khi service trở lại), graceful degradation (agent vẫn chạy standalone khi dependency chết). (https://github.com/dhyansraj/mcp-mesh — README "Resilience")
- Discovery pull-based + heartbeat re-resolution: dependency có thể nằm ở process khác, máy khác, ngôn ngữ khác — "viết hàm local, được hệ phân tán". (https://github.com/dhyansraj/mcp-mesh — README "Key Features: DDDI")
- LLM cũng là một dependency injectable (`@mesh.llm`); có auto-failover giữa các provider (vd: Claude agent chết → mesh tự chuyển sang API thường, hồi phục thì chuyển về). (https://github.com/dhyansraj/mcp-mesh — README ví dụ TripPlanner)
- Quản trị bằng `meshctl` (kiểu kubectl), quan sát bằng Grafana/Tempo/Redis theo stack chuẩn.

## Yêu cầu triển khai (ngắn gọn)
- Có 2 tầng triển khai: **nhẹ** — code chạy local/Docker Compose ("same code, zero changes"); **nặng** — K8s-native qua Helm chart, kèm observability stack (Grafana, Tempo, Redis). (https://github.com/dhyansraj/mcp-mesh — README "For Developers", "For DevOps & Platform Teams")
- Không yêu cầu GPU (LLM gọi qua provider API); agent Python/TS chạy được trên máy yếu.
- Mức độ chín muồi: **khá tốt trong nhóm repo nhỏ** — 814 commit, 147 release, license MIT rõ ràng, có docs site (mcp-mesh.ai), Discord, ví dụ và deployment patterns. (https://github.com/dhyansraj/mcp-mesh — "Repository details")

## Verdict — Mạnh deploy được không?
**Được có điều kiện** — (1) Tầng K8s + observability stack (Grafana/Tempo/Redis) là **quá nặng** cho VPS Oracle 1GB RAM và điện thoại; chỉ nên thử trên **A1 Flex 12GB** nếu Mạnh dựng k3s nhẹ, hoặc bỏ luôn tầng K8s. (2) Tầng nhẹ (agent Python + Docker Compose, không K8s) thì **deploy được** trên hạ tầng hiện tại qua Tailscale, nhưng chưa xác minh được agent Python thuần có chạy ổn mà không cần registry/mesh control-plane không — cần test thực tế. Điểm cộng lớn cho track của Mạnh: triết lý DDDI/auto-failover rất gần với thứ conductor MVP đang làm (capability routing + chống fake), nên dù không deploy nguyên si thì cũng là **chất liệu thiết kế tốt** cho blueprint.

## Nguồn (URL cụ thể đã đọc)
- https://github.com/dhyansraj/mcp-mesh (README đầy đủ — quick start, DDDI, resilience, observability, K8s-native; 42 stars, 814 commit, 147 release, license MIT)
