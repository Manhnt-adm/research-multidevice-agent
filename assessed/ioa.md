# IoA (Internet of Agents)
- URL: https://github.com/OpenBMB/IoA
- Stars: 836 (tại thời điểm đọc)
- Đánh giá ngày: 23/10/2026

## Là gì (2 dòng)
Framework mã nguồn mở của OpenBMB cho các AI agents hợp tác phân tán, kết nối kiểu "internet-like": các agents khác nhau (AutoGPT, Open Interpreter, ReAct...) tự lập team lồng nhau để giải task phức tạp. License Apache 2.0; kiến trúc chia im_server (server), im_client (client), agent images, Milvus vector DB. (https://github.com/OpenBMB/IoA)

## Kiến trúc multi-device (chi tiết: distributed thực sự đa máy hay chỉ multi-runtime)
- README có dòng riêng: "Want to run IoA across different devices? Check out our distributed setup guide" (https://github.com/OpenBMB/IoA) — tức tác giả **tuyên bố** hỗ trợ chạy xuyên devices.
- BẢN CHẤT từ Quick Start: agents là **Docker images** (weize/ioa-server, ioa-client, react-agent, autogpt, open-interpreter) chạy trên docker network `agent_network`, giao tiếp qua server trung tâm (endpoint `http://127.0.0.1:5050/launch_goal`). Đây là distributed theo kiểu container-cluster qua mạng — về nguyên tắc multi-machine được, nhưng agents là container Docker.
- Chi tiết distributed setup guide (openbmb.github.io, link từ README) **chưa đọc được** do browser bị rate-limit — chưa xác minh được: agent có join từ máy không chạy Docker được không, discovery/auth giữa các máy ra sao.

## Yêu cầu triển khai
- Docker bắt buộc (prerequisites duy nhất nêu trong README). (https://github.com/OpenBMB/IoA)
- Milvus vector DB chạy qua docker-compose (`dockerfiles/compose/milvus.yaml`) — thêm một stack nặng. (https://github.com/OpenBMB/IoA)
- `.env` yêu cầu **điền OpenAI API key** (trả phí) — không thấy đề cập OpenRouter/base-url tuỳ biến trong README (https://github.com/OpenBMB/IoA).
- Repo chỉ có **21 commits** (tạo 07/2024) — dấu hiệu project demo nghiên cứu, ít được duy trì tiếp.
- Điện thoại chạy Termux **không chạy Docker** được theo cách thông thường → agents dạng container không thể chạy trên S8+/S20FE.

## Docs có đủ để dựng không?
README có Quick Start 6 bước khá chi tiết cho single-machine (pull image, compose Milvus, launch). Nhưng phần quan trọng nhất với track này — distributed setup guide — chưa đọc được (rate limit). Có link tới paper arXiv và docs site openbmb.github.io. Kết luận: đủ để dựng demo 1 máy, **chưa đủ để đánh giá triển khai đa máy**.

## Verdict — Mạnh deploy được không?
**Loại** — 3 lý do: (1) agents là Docker containers + Milvus: không chạy được trên 2 điện thoại Termux, và quá nặng cho VPS 1GB (A1 Flex 12GB may ra gánh được demo 1 máy); (2) yêu cầu OpenAI API key trả phí, trái điều kiện "chi phí ~0" — khả năng trỏ sang OpenRouter free **chưa xác minh được**; (3) repo 21 commits từ 2024, kiểu demo học thuật, rủi ro bỏ rơi cao. Nếu chỉ muốn nghịch demo single-server trên A1 thì được, nhưng không đáp ứng mục tiêu multi-device của Mạnh.

## Nguồn (URL cụ thể đã đọc)
- https://github.com/OpenBMB/IoA (README + metadata repo: stars, license Apache-2.0, 21 commits)
- Distributed setup guide (openbmb.github.io, link từ README): chưa đọc được — browser rate-limit 429 tại thời điểm đánh giá.
