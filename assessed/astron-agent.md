# astron-agent (iflytek)

- URL: https://github.com/iflytek/astron-agent
- Stars: 8,915 (tại thời điểm đọc)
- Đánh giá ngày: 23/10/2026

## Là gì (2 dòng)
Nền tảng **agentic workflow** doanh nghiệp (Apache 2.0): xây agent/workflow bằng low-code, tích hợp model, MCP tools, RPA và cộng tác nhóm — hướng "xây SuperAgent chạy production", deploy on-premise HA. Không phải orchestrator điều phối agent trên nhiều thiết bị.

## Kiến trúc multi-device / vai trò trong fleet (chi tiết vừa đủ)
- Kiến trúc là **platform tập trung**: workflow được gọi từ project ngoài qua HTTP/SSE ("call published workflows from an existing project via HTTP/SSE" — Integration Guide). Tức agent/workflow chạy trên server trung tâm, thiết bị khác chỉ gọi API — khác với mô hình fleet của Mạnh (agent chạy thật trên từng máy, conductor điều phối). (https://github.com/iflytek/astron-agent, mục README "Documentation")
- Hai dự án anh em: **SkillHub** (registry skill — đánh giá riêng) và **AstronRPA** (suite RPA gọi trực tiếp như node trong workflow). (https://github.com/iflytek/astron-agent, mục "Related Projects")
- Auth qua Casdoor (gateway xác thực riêng, mặc định `admin/123` phải đổi), frontend qua nginx proxy, deploy bằng `docker-compose-with-auth.yaml` trong `docker/astronAgent`; Helm chart đang "under development". (https://github.com/iflytek/astron-agent, mục "Quick Start")
- Hệ sinh thái nghiêng về iFLYTEK: "Ready-to-Use Tool Ecosystem" lấy từ iFLYTEK Open Platform (xfyun.cn), có bản cloud dùng sẵn tại agent.xfyun.cn, docs dùng (xfyun.cn, feishu) chủ yếu tiếng Trung, danh sách adopters toàn doanh nghiệp Trung Quốc (中国电信, 东华软件...). Listed trong CNCF Landscape. (https://github.com/iflytek/astron-agent, mục "Key Features", "Adopters", "Documentation")
- Không tìm thấy cơ chế device enrollment / agent chạy phân tán trên điện thoại / mesh như rook hay conductor — vai trò multi-device của nó là "server trung tâm + client gọi API", không phải điều phối fleet.

## Yêu cầu triển khai
- Docker Compose, nhiều service (app, auth Casdoor, nginx...), file `.env` cấu hình; có hướng dẫn DEPLOYMENT_GUIDE_WITH_AUTH riêng. Mức RAM/CPU tối thiểu **chưa xác minh được** trong phần đã đọc, nhưng với stack "enterprise HA" nhiều service thì khó chạy vừa VPS 1GB của Mạnh.

## Docs có đủ để dựng không?
Có vẻ đủ: Deployment Guide, Configuration, Quick Start, Development Guide, Tutorial, Best Practices, FAQ — nhưng phần lớn host trên xfyun.cn/feishu (tiếng Trung), Quick Start tiếng Anh trong README còn sơ sài (Helm chưa có). (https://github.com/iflytek/astron-agent, mục "Documentation" và "Quick Start")

## Verdict — có dùng được cho Mạnh không? (Được / Được có điều kiện / Loại + 1–3 dòng lý do)
**Loại.** (1) Đây là nền tảng workflow doanh nghiệp tập trung, không phải orchestrator cho agent chạy trên nhiều thiết bị — chồng chéo và thừa so với MVP conductor Mạnh đã tự build. (2) Stack nặng nhiều service (Casdoor, nginx, HA), không hợp triết lý chi phí ~0 của Mạnh. (3) Hệ sinh thái gắn chặt iFLYTEK Open Platform/Astron Cloud Trung Quốc — nguy cơ phụ thuộc nhà cung cấp, docs chủ yếu tiếng Trung.

## Nguồn (URL cụ thể đã đọc)
- https://github.com/iflytek/astron-agent (README + repo tree, stars/license/topics/quick start)
