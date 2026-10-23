# hatchet
- URL: https://github.com/hatchet-dev/hatchet
- Stars: 8169
- Đánh giá ngày: 23/10/2026
## Là gì (2 dòng)
Hatchet là orchestration engine mã nguồn mở (MIT, engine viết bằng Go) cho background task, AI agent và durable workflow — chạy được queue + DAG + durable execution trên cùng một nền tảng. Có SDK cho Python, TypeScript, Go, Ruby; cung cấp retries (exponential backoff), cron job/scheduled runs, task routing, dashboard quan sát realtime và OpenTelemetry/Prometheus.

## Kiến trúc multi-device
Mô hình **control plane tập trung + worker phân tán**: control plane gồm API server (REST), Engine (gRPC, task scheduling), Postgres (lớp durable lưu trạng thái task + observability) và Dashboard; **workers là long-running process nằm ở bất kỳ máy nào**, kết nối về engine bằng **gRPC stream hai chiều long-lived** do worker chủ động khởi tạo (Dispatcher service: Register → Heartbeat liên tục → Listen stream nhận action START_STEP_RUN; rớt mạng tự reconnect, mất heartbeat thì engine đánh dấu worker inactive và giao việc sang worker khác). Worker định nghĩa trong code (ví dụ `hatchet.worker("image-worker", slots=4, workflows=[...])`), giới hạn tải bằng `slots`, điều hướng bằng worker labels/affinity. Có thêm chế độ **Webhook Worker** cho môi trường không giữ kết nối dài (engine push HTTP + xác thực HMAC) — nhưng mặc định workers là process long-lived. Vì kết nối do worker chủ động mở (client-initiated, TLS), worker trên máy Mạnh (VPS, S8+ qua Termux) trỏ về control plane qua Tailscale là khả thi về mặt mạng.

## Yêu cầu triển khai
- **BẮT BUỘC: PostgreSQL** (15.6 trong docs) — lưu trạng thái workflow, task run, observability. 
- RabbitMQ: **không bắt buộc** — docs cho phép `SERVER_MSGQUEUE_KIND=postgres` để dùng Postgres làm message broker, bỏ RabbitMQ.
- Cách chạy: (1) `hatchet-lite` — một image Docker duy nhất gồm engine + API (docs ghi cho dev/low-throughput); (2) Docker Compose production gồm postgres + (rabbitmq) + hatchet-engine + hatchet-dashboard + migration + setup-config; (3) Helm chart cho Kubernetes.
- Docs không công bố con số RAM tối thiểu; bản chất: Postgres (max_connections=1000 trong compose mẫu) + engine Go + dashboard chạy cùng một máy.

## Docs có đủ để dựng không?
Có. docs.hatchet.run rất đầy đủ: trang self-hosting (lite / docker compose / helm), CLI quickstart, cookbooks chạy task đầu tiên, 4 SDK có ví dụ được test, trang so sánh với Temporal/DBOS/Celery/Airflow. Hướng dẫn docker-compose copy-paste được (tạo tenant, API token cho worker đều có sẵn).

## Verdict — Mạnh deploy được không?
**Không nên đặt control plane lên VPS 1GB** — stack gồm Postgres + engine + dashboard là quá nặng cho máy e2 micro (kể cả hatchet-lite + Postgres gộp, và Oracle free tier còn phải gánh các agent đang chạy). Nếu muốn thử thật, đặt control plane lên **A1 Flex (12GB) khi Mạnh dựng xong**, còn worker Python chạy qua Termux trên S8+ hoặc VPS nhỏ trỏ về qua Tailscale (worker nhẹ, chỉ cần Python + SDK + token). Lựa chọn rẻ hơn cho Mạnh: dùng Hatchet Cloud free trial trước để đánh giá, khỏi tự host ngay. Hatchet đúng "dòng máu" cần cho điều phối agent đa thiết bị (worker phân tán + durable + retries + cron đều có sẵn, MIT không phí bản quyền), nhưng chi phí hạ tầng tự host là rào cản thật.

## Nguồn
- https://github.com/hatchet-dev/hatchet (README: mô tả, stars 8169, MIT, tính năng, so sánh với Temporal/DBOS/Celery/Airflow)
- https://docs.hatchet.run (What is Hatchet: khái niệm Task/Worker/Durable Workflow, Postgres là infra dependency duy nhất cho workload đơn giản, 100% MIT)
- https://docs.hatchet.run/self-hosting (các thành phần control plane: API server, Engine gRPC, Postgres bắt buộc, RabbitMQ optional, Dashboard; workers kết nối từ xa được)
- https://docs.hatchet.run/self-hosting/hatchet-lite (hatchet-lite single image dev/low-throughput)
- https://docs.hatchet.run/self-hosting/docker-compose (compose mẫu: postgres 15.6, rabbitmq, engine, dashboard; SERVER_MSGQUEUE_KIND=postgres để bỏ RabbitMQ)
- https://docs.hatchet.run/v1/from-celery-to-hatchet (worker định nghĩa trong code, slots, token HATCHET_CLIENT_TOKEN)
- https://deepwiki.com/hatchet-dev/hatchet-typescript/4.2-dispatcherclient-and-actionlistener (DispatcherClient: Register/Heartbeat; ActionListener giữ long-lived gRPC stream, tự retry khi rớt; webhook worker + HMAC)
- https://deepwiki.com/hatchet-dev/hatchet/3.4-dispatcher-system (Dispatcher duy trì persistent gRPC connections với workers, điều phối bidirectional)
