# golem (Golem Cloud OSS)
- URL: https://github.com/golemcloud/golem
- Stars: 1516 (tại thời điểm đọc, 23/10/2026)
- Đánh giá ngày: 23/10/2026

## Là gì (2 dòng)
Golem là platform mã nguồn mở chạy WebAssembly component theo mô hình "durable execution": mỗi agent là một **worker** sống lâu, có state, tự phục hồi sau crash/restart/nâng cấp nhờ ghi lại oplog — không mất state, không chạy trùng công việc.
Hỗ trợ viết agent bằng Rust, TypeScript, Scala, MoonBit; tương tác qua HTTP/gRPC API và Golem CLI.

## Kiến trúc multi-device (chi tiết: distributed như thế nào, worker/agent chạy ở đâu)
- 6 service lõi phối hợp với nhau (https://deepwiki.com/golemcloud/golem/2.1-worker-execution-system):
  - **Worker Service** — gateway API (REST/gRPC/WebSocket): tạo worker, gọi hàm, quản lý vòng đời.
  - **Worker Executor** — runtime WASM (Wasmtime): thực thi worker, ghi oplog để durable execution.
  - **Shard Manager** — phân phối worker qua các executor, xử lý failover và tái gán worker khi executor chết.
  - **Component Service** + **Component Compilation Service** — lưu/trình biên dịch WASM component (SQLite `components.db`, blob storage).
  - **Cloud Service** — multi-tenancy, account/project/auth (SQLite hoặc Postgres).
- Distributed theo nghĩa **server-side cluster**: chạy nhiều worker executor (tham số `WORKER_EXECUTOR_REPLICAS`), shard manager định tuyến worker tới executor đúng; worker không có trong RAM sẽ được load lại và khôi phục state trong suốt (https://golem.cloud/blog/durable-execution-is-not-just-for-failures/).
- **Agent chạy ở đâu:** agent = worker = instance của WASM component, chạy TRONG executor của cluster. Điện thoại/máy khác chỉ đóng vai trò **client** gọi API (HTTP/gRPC/CLI), không phải node thực thi. Việc chạy executor trên điện thoại (ARM) chưa xác minh được.

## Yêu cầu triển khai (cần cài gì — đặc biệt kiểm tra có cần cluster/K8s/nặng không)
- **Không cần K8s.** Có 2 cấu hình Docker Compose chính thức (https://github.com/golemcloud/golem/tree/main/docker-examples):
  - **single-node**: image all-in-one `golemservices/golem` + Caddy làm TLS; 1 server duy nhất, SQLite mặc định (có Litestream backup, hoặc chuyển Postgres qua `GOLEM_POSTGRES_HOST`), S3-compatible bucket cho blob. Chạy trên 1 host, port 9881.
  - **production-cluster**: tách từng service, nhiều worker executor, Postgres + S3 ngoài host, Caddy TLS — vẫn Docker Compose, không bắt buộc K8s.
- Quickstart chính thức: cài Docker + Docker Compose, `docker compose up -d` là chạy (https://learn.golem.cloud/docs/quickstart).
- Yêu cầu hạ tầng đáng chú ý: single-node cần S3 bucket (có thể tự host MinIO — chưa xác minh chi tiết trong docs đã đọc) và Postgres cho production; image server chạy dưới uid/gid 65534. Không thấy yêu cầu RAM tối thiểu công bố trong các trang đã đọc.

## Docs có đủ để dựng không? (đánh giá trung thực)
Đủ cho single-node: quickstart (https://learn.golem.cloud/docs/quickstart) + README ví dụ trong `docker-examples/single-node` và `production-cluster` hướng dẫn từng bước `cp .env.example .env` → `docker compose up -d`. Docs kiến trúc sâu nằm trong repo/docs (Nextra site tại learn.golem.cloud). Thiếu: không tìm thấy số liệu RAM/CPU tối thiểu chính thức, và chưa xác minh image có bản ARM64 không (quan trọng cho A1 Flex ARM và điện thoại).

## Verdict — Mạnh deploy được không? Được có điều kiện
Deploy được trên A1 Flex (12GB) ở chế độ single-node all-in-one Docker — không cần K8s, không subscription (bản OSS tự host). NHƯNG: (1) Golem là platform để **viết agent bằng WASM component**, không phải công cụ chạy sẵn Antigravity/OpenClaw agent trên nhiều máy — muốn dùng phải tự code agent (Rust/TS) gọi LLM qua HTTP, công sức lớn; (2) điện thoại chỉ làm client gọi API, không thành node thực thi (executor ARM chưa xác minh); (3) chưa rõ image có chạy ARM64 không — A1 Flex của Mạnh là ARM, cần kiểm chứng trước khi dựng.

## Nguồn (liệt kê URL cụ thể đã đọc)
- https://github.com/golemcloud/golem (README, root files, stars)
- https://github.com/golemcloud/golem/tree/main/docker-examples (danh sách single-node / production-cluster / distributed-etcd)
- https://github.com/golemcloud/golem/tree/main/docker-examples/single-node (all-in-one, Litestream, port 9881)
- https://github.com/golemcloud/golem/tree/main/docker-examples/production-cluster (cluster, WORKER_EXECUTOR_REPLICAS, shard manager 1 replica)
- https://learn.golem.cloud/docs/quickstart (quickstart, CLI, tạo worker, invoke)
- https://deepwiki.com/golemcloud/golem/2.1-worker-execution-system (6 service lõi, worker service API)
- https://deepwiki.com/golemcloud/golem (kiến trúc service, oplog, shard manager)
- https://golem.cloud/blog/durable-execution-is-not-just-for-failures/ (durable execution, worker idle restore, 10000 workers)
- https://golem.cloud/blog/the-golem-forge-hackathon/ (agent chạy local, durable promises/scheduling — context bổ sung)
