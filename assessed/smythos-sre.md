# SmythOS/sre
- URL: https://github.com/SmythOS/sre
- Stars: 1303 (tại thời điểm đọc)
- Đánh giá ngày: 23/10/2026

## Là gì (2 dòng)
SmythOS Runtime Environment (SRE) — runtime + SDK + CLI mã nguồn mở cho AI agents, tự gọi là "The Linux of AI Agents" (lớp OS trừu tượng hoá LLM, VectorDB, storage, cache qua API thống nhất). Monorepo TypeScript gồm 3 package: `packages/core` (SRE kernel), `packages/sdk`, `packages/cli`; license MIT. (https://github.com/SmythOS/sre)

## Kiến trúc multi-device (chi tiết: distributed thực sự đa máy hay chỉ multi-runtime)
- README khẳng định agents chạy được "across local, cloud, and edge environments" (https://github.com/SmythOS/sre), và nguyên tắc "Development to Production Evolution": cùng một business logic, đổi connector (Local → S3/Redis/...) là chạy ở hạ tầng khác mà không sửa code.
- BẢN CHẤT: đây là **multi-runtime** (cùng code chạy ở nhiều môi trường) chứ README không mô tả một mặt lưới (mesh) mà trong đó một cụm agents phân tán trên nhiều máy vật lý tự phối hợp với nhau. Không có mô tả nào về coordinator/gossip/federation giữa các node SRE trên các máy khác nhau.
- Docs chi tiết (SRE Core Documentation tại smythos.github.io, link từ README) chưa đọc được do browser bị rate-limit khi mở — phần "distributed thực sự đa máy vật lý" vì vậy **chưa xác minh được**.

## Yêu cầu triển khai
- Node.js + npm/pnpm: cài CLI bằng `npm i -g @smythos/cli`, scaffold project bằng `sre create`; hoặc `npm install @smythos/sdk` vào project có sẵn. (https://github.com/SmythOS/sre)
- Connectors mặc định khi dev: Cache=RAM, Storage=Local, Log=ConsoleLog — tức chạy được ngay trên máy nhỏ không cần dịch vụ ngoài. (https://github.com/SmythOS/sre)
- LLM connectors chính thức: OpenAI, Anthropic, Google AI, AWS Bedrock, Groq, Perplexity — **không thấy OpenRouter/OpenAI-compatible generic trong danh sách** (https://github.com/SmythOS/sre). (Groq có free tier — chưa xác minh quota cụ thể.)
- Repo có 1.970 commits, vẫn đang hoạt động.

## Docs có đủ để dựng không?
README có Quick Start đầy đủ (CLI + SDK), link tới SDK Documentation và SRE Core Documentation (smythos.github.io) — chưa đọc được do rate limit nên chưa đánh giá được độ sâu của docs nâng cao (deploy production, clustering). Quick Start cơ bản: đủ.

## Verdict — Mạnh deploy được không?
**Được có điều kiện** — Dùng được như agent runtime/SDK trên A1 Flex (Node nhẹ, chạy local không cần dịch vụ ngoài nhờ connector RAM/Local). Điều kiện: (1) "distributed" của nó là multi-runtime, khả năng orchestrate agents xuyên nhiều máy vật lý **chưa xác minh được** — không thay thế được coordinator trong kiến trúc 3 tầng; (2) connector OpenRouter chưa có trong danh sách chính thức, cần xác minh thêm có hỗ trợ custom endpoint OpenAI-compatible không; (3) chưa đọc docs production nên chưa rõ yêu cầu RAM thực tế của SRE khi chạy nhiều agents.

## Nguồn (URL cụ thể đã đọc)
- https://github.com/SmythOS/sre (README + metadata repo: stars, license MIT, commits)
- Docs gốc (smythos.github.io, link từ README): chưa đọc được — browser rate-limit 429 tại thời điểm đánh giá.
