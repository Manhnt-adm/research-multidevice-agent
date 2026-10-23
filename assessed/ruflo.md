# Ruflo
- URL: https://github.com/ruvnet/ruflo
- Stars: 78.547 (qua GitHub API, tại thời điểm đọc 23/10/2026; 9.235 fork, 966 issue mở, push mới nhất 23/10/2026 — repo đang phát triển rất mạnh)
- Đánh giá ngày: 23/10/2026

## Là gì (2 dòng)
Ruflo (tên cũ Claude Flow, của rUv) là **meta-harness cho Claude Code và Codex**: lớp thực thi quanh model, thêm 100+ agent chuyên biệt, swarm phối hợp, memory tự học, comms liên máy (federation) và guardrail bảo mật. Tóm tắt của chính repo: "Agent = Model + Harness. Ruflo is the harness — the execution layer around Claude Code and Codex."

## Kiến trúc multi-device (chi tiết)
Federation được tài liệu hóa đầy đủ trong `docs/federation/README.md` và có code thật ở `v3/@claude-flow/plugin-agent-federation/src/` (thư mục `domain/`, `transport/`, `mcp-tools.ts`, test riêng, 8 ADR từ ADR-097 đến ADR-111):

- **Identity**: mỗi node giữ cặp key **Ed25519** (lưu `keys-<nodeId>.json`, mode 0600); peer trao đổi manifest ký bằng Ed25519, không có directory trung tâm.
- **Thang trust 5 bậc**: `UNTRUSTED → VERIFIED → ATTESTED → TRUSTED → PRIVILEGED`. Trust tăng dần qua tương tác thành công; mỗi bậc mở thêm capability (`discovery`, `send`, `share-context`, `remote-spawn`, `full-memory`…).
- **Transport thật**: WSS với permessage-deflate, multiplexing, **optional cert pinning** (ADR-107). ⚠️ Cần đính chính: repo KHÔNG dùng mTLS làm mặc định — ADR-107 ghi rõ cert lifecycle là "operator policy (Let's Encrypt? internal CA? **mTLS w/ pinned self-signed? all valid choices**)" và tài liệu ghi **tailnet-WG là đường được khuyến nghị**. mTLS chỉ là một lựa chọn, không phải tính năng có sẵn.
- **WG mesh opt-in** (alpha.14, ADR-111): khi bật, trust change truyền xuống L3 — peer bị SUSPEND thì `wg set … allowed-ips ""`, bị EVICT thì remove; có witness chain ký Ed25519 append-only.
- **Anti-goal ghi rõ**: "NAT traversal magic" — KHÔNG tự xuyên NAT, bắt buộc dùng **Tailscale (hoặc Headscale)**; federation ride on top. Rất hợp với mesh Tailscale của Mạnh.
- **Cross-machine thật sự làm gì**: gửi envelope có chữ ký giữa các cài đặt Ruflo (agent-handoff, query đồng bộ, proposal consensus), audit + PII pipeline + budget/token per-peer + circuit breaker (ACTIVE → SUSPENDED → EVICTED). `remote-spawn` (spawn agent từ xa) chỉ mở ở bậc PRIVILEGED — tức là có, nhưng ở bậc trust cao nhất và cần bootstrap out-of-band.
- Tình trạng: federation ở **alpha** (`1.0.0-alpha.14`, alpha.15 "in flight"); plugin cài qua `npx ruflo plugins install @claude-flow/plugin-agent-federation`, endpoint mặc định `ws://<host>:9100`.
- Claim "swarm tới 100 coding agent": **chưa xác minh được** — không thấy con số 100 agent trong README/USERGUIDE/docs đã đọc.

## Yêu cầu triển khai
- Node.js + `npx ruflo` (package `ruflo` trên npm, version 3.59.17 tại thời điểm đọc). Có 2 đường: plugin Claude Code (cần **Claude Code ≥ 2.1.287**, tức phụ thuộc subscription Anthropic) hoặc CLI đầy đủ (`npx ruflo init` → daemon, 98 agent, MCP server `npx ruflo mcp start`, TUI `npx ruflo tui`).
- Federation: cài plugin, init node (sinh Ed25519 keypair), `federation_join` tới endpoint peer — peer phải reach được nhau (qua Tailscale/LAN; không tự xuyên NAT).
- RAM tối thiểu: **chưa xác minh được** (docs không nêu con số).

## Docs có đủ để dựng không?
Đầy đủ và thừa: README marketing-heavy nhưng `docs/federation/README.md` là user guide thực thụ (quickstart từng bước, bảng MCP tool, trust ladder, circuit breaker, bringup WG mesh `docs/federation/phase7-mesh-bringup.md`), ADRs riêng từng quyết định thiết kế. Có thể dựng được theo docs.

## Verdict — Mạnh deploy được không?
**Được có điều kiện.** Federation cross-machine là có thật (code + ADR + test, Ed25519 + WSS/pinning, trust ladder, ride trên Tailscale) — phù hợp mesh của Mạnh. Điều kiện: (1) còn **alpha** (1.0.0-alpha.14/15) — chờ ổn định hoặc chấp nhận bug; (2) Ruflo là harness xoay quanh **Claude Code/Codex**, không phải executor đa thiết bị generic — Mạnh dùng model free qua OpenRouter, tương thích chưa được xác minh trong docs đã đọc (có plugin `ruflo-deepseek-harness` gợi ý đa-provider nhưng chưa kiểm chứng); (3) repo nặng, 966 issue mở, phát triển rất nhanh — rủi ro đổi interface; (4) mạnh nhất cho use case code-agent, không phải điều khiển điện thoại (rook ở đợt 2 vẫn phù hợp hơn cho tầng S8+/S20FE).

## Nguồn (URL cụ thể đã đọc)
- https://github.com/ruvnet/ruflo (README: tagline "An agent meta-harness for Claude Code and Codex", get-started, quick start 2 install path, origin/claude-flow lineage)
- https://github.com/ruvnet/ruflo/blob/main/docs/federation/README.md (federation user guide: Ed25519 identity, trust ladder 5 bậc, circuit breaker, WSS transport, ADR-111 WG mesh, MCP tools, anti-goal NAT/Tailscale, version alpha.14)
- https://github.com/ruvnet/ruflo/blob/main/v3/docs/adr/ADR-107-federation-tls.md (mTLS chỉ là "operator policy choice", tailnet-WG là đường khuyến nghị)
- https://github.com/ruvnet/ruflo/tree/main/v3/%40claude-flow/plugin-agent-federation/src (xác minh code federation tồn tại: domain/, transport/, mcp-tools.ts, __tests__)
- https://api.github.com/repos/ruvnet/ruflo (stars 78.547, fork 9.235, open issues 966, pushed_at 2026-10-23)
