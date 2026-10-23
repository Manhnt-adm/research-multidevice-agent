# agentscope-java
- URL: https://github.com/agentscope-ai/agentscope-java
- Stars: 6091 (tại thời điểm đọc, commit main 7976a20)
- Đánh giá ngày: 23/10/2026

## Là gì (2 dòng)
Framework Java (nhà Alibaba, dùng nội bộ ở 13+ đơn vị Alibaba Group) để xây agent AI chạy production theo kiểu "distributed, long-running, enterprise-grade": gồm Harness SDK (nhúng agent vào app Java/Spring Boot) và AgentScope Service — một Control Plane tự host (Agent-as-a-Service) quản lý vòng đời, session, dashboard cho các agent. Hoàn toàn là server-side; KHÔNG phải công cụ điều khiển điện thoại hay fleet thiết bị.

## Kiến trúc multi-device (chi tiết: orchestrator nói chuyện với device/node bằng gì — protocol, message, ai dial ai, state ở đâu)
Chữ "distributed" ở đây = multi-replica trên server, không phải multi-device (điện thoại):
- **Parent → child subagent:** trong cùng process là instance `HarnessAgent` cục bộ; qua mạng thì parent khai báo subagent bằng `url(...)` và **parent chủ động dial HTTP** tới remote agent qua "Agent Protocol" (REST + SSE streaming, có endpoint `POST /tasks/{id}/resume` cho HITL); sync (`timeout_seconds>0`) hay background (`=0`) đều được; streaming sự kiện remote forward về parent qua SSE.
- **State:** không nằm trên node mà nằm ở store dùng chung — `AgentStateStore` (Redis / MySQL / PostgreSQL / JSON file / in-memory) cho session state, `BaseStore` (Redis/JDBC) cho workspace dùng chung (MEMORY.md, skills, sessions), snapshot sandbox trên Redis/OSS. Nhờ đó bất kỳ replica nào cũng resume được session của replica khác (cross-replica session recovery, CAS chống conflict).
- **Task background:** state task ghi vào store dùng chung (mọi node đều đọc được) nhưng **execution pin vào node đã tạo**; node khác chỉ đọc kết quả hoặc cancel (executing node poll cancel flag).
- **Không có:** khái niệm phone/device/node vật lý, ADB, enrollment thiết bị, hay agent chạy trên Android. Các "Channel" (DingTalk, Feishu, WeCom) là integration nhắn tin doanh nghiệp Trung Quốc, không phải Telegram/Zalo của Mạnh.

## Yêu cầu triển khai (cần cài gì, phụ thuộc nặng nhẹ, phần cứng yêu cầu, Java version...)
- **Harness SDK:** JDK 17+ (bắt buộc), Maven, model provider (mặc định DashScope — cần `DASHSCOPE_API_KEY` trả phí; có module OpenAI-compatible `agentscope-extensions-model-openai` hỗ trợ custom endpoint, OpenRouter chưa được docs nêu tên — chưa xác minh được).
- **Distributed thật:** Redis (khuyến nghị, qua `agentscope-extensions-redis`) hoặc MySQL/PostgreSQL cho state store; Docker/Kubernetes cho sandbox (5 loại sandbox: Docker, K8s, Daytona, E2B, AgentRun).
- **Service Control Plane:** Docker Compose (local eval) hoặc Helm chart lên Kubernetes (production), cần database (Postgres được khuyến nghị) + model API key.
- Với hạ tầng của Mạnh: JDK 17 trên Termux (Android) cực khó/không thực tế; JVM + Spring Boot + Redis/Postgres + Docker vượt xa VPS Oracle 1GB RAM và không ăn nhập gì với phone farm qua ADB.

## Docs có đủ để dựng không? (đánh giá trung thực độ đầy đủ docs)
Docs rất đầy đủ và chuyên nghiệp (Mintlify tại java.agentscope.io, song ngữ Anh–Trung, v1/v2): quickstart, kiến trúc Harness, subagent (local/remote), going-to-production (bảng so sánh single-node vs distributed, validation chain), Service deployment (Docker Compose eval → Helm production), API reference. Đủ để dev Java dựng được — nhưng toàn bộ docs viết cho scenario enterprise server, không có trang nào hướng dẫn chạy trên thiết bị di động hay điều phối phone.

## Verdict — Mạnh deploy được không?
**Loại** — Đây là framework xây agent doanh nghiệp chạy trên server/K8s của Alibaba, không phải hệ điều phối đa thiết bị: không có agent-on-phone, không ADB/Termux, không enrollment thiết bị; JDK 17 + Redis/Postgres + Docker/K8s vượt xa ngân sách ~0 và hạ tầng của Mạnh, trong khi "remote subagent" qua HTTP chỉ là node-server gọi node-server chứ không giải quyết bài toán phone farm mini (S8+/S20FE) của anh.

## Nguồn (liệt kê URL cụ thể đã đọc: repo, file docs...)
- https://github.com/agentscope-ai/agentscope-java (README: "Build distributed, production-grade, long-running agents", 6091 stars, JDK 17+, Apache-2.0, các module agentscope-core/harness/service)
- https://java.agentscope.io (landing: Harness SDK vs Service, self-hosting recommended, Alibaba Group dùng nội bộ)
- https://java.agentscope.io/v2/en/docs/harness/subagent (remote subagent qua `url` + Agent Protocol HTTP/SSE, `POST /tasks/{id}/resume`, background task pin ở node tạo, state trong store dùng chung)
- https://java.agentscope.io/v2/en/docs/others/going-to-production (`DistributedStore` một dòng, AgentStateStore Redis/MySQL/PostgreSQL, BaseStore, sandbox Docker/K8s/E2B/AgentRun, IsolationScope)
- https://java.agentscope.io/v2/en/service/index (Service = self-hostable Agent-as-a-Service, Docker Compose eval, Helm chart cho K8s production, External/Hosted Agent)
- https://java.agentscope.io/v2/en/service/quickstart (Deploy Service: Docker Engine + Compose v2 + database + DashScope API key)
- https://java.agentscope.io/v2/en/integration/model/openai (`agentscope-extensions-model-openai` dùng cho OpenAI-compatible endpoints + custom endpoint qua builder; OpenRouter không được nêu tên)
- https://www.alibabacloud.com/blog/agentscope-java-2-0-building-a-distributed-enterprise-grade-foundation-for-ai-agents_603231 (giải thích distributed session/sandbox management qua Redis/object store)
- https://github.com/agentscope-ai/agentscope-java/tree/main/docs (cấu trúc docs Mintlify)
- https://raw.githubusercontent.com/agentscope-ai/agentscope-java/main/docs/docs.json (navigation: các trang architecture, session store, protocol A2A/AG-UI, channel DingTalk/Feishu/WeCom)
