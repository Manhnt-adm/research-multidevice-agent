# PantheonOS

- URL: https://github.com/aristoteleo/PantheonOS
- Stars: 490 (tại thời điểm đọc)
- Đánh giá ngày: 23/10/2026

## Là gì (2 dòng)

Framework multi-agent (Python) cho data science, điểm nhấn là "evolvable" (agent tự cải tiến code bằng genetic algorithm) và distributed qua NATS; đi kèm subsystem **Pantheon-Fleet** (Go) cho phép agent chạy lệnh trực tiếp trên một pool máy. (https://github.com/aristoteleo/PantheonOS — README L178–L197)

## Kiến trúc multi-device (chi tiết)

Có 2 tầng distributed khác nhau, cần phân biệt rõ:

1. **Distributed execution (Python, NATS):** NATS server làm message broker/coordinator trung tâm; client (REPL/UI/API) gửi yêu cầu, các **worker** (`python -m pantheon.endpoint --mode worker`, có thể giới hạn toolset theo máy: file_manager / python_interpreter / shell) chạy trên máy khác nhau và thực thi tool. NATS tự load-balance giữa worker; hỗ trợ NATS cluster 3 node cho HA, auth/token/TLS. Docs còn cho sẵn docker-compose và manifest Kubernetes. (https://pantheonos.readthedocs.io/en/latest/advanced/distributed.html — toàn trang)
2. **Pantheon-Fleet (Go, riêng biệt):** runner binary mỗi node chạy `fleet up` → join registry (NATS + JetStream KV), nhận task và **chạy shell/python trực tiếp trên node qua subprocess**, truyền file node↔node bằng libp2p (QUIC, chunked + sha256 — đã verify 50MB @ ~200MB/s). Agent điều khiển qua `python/fleet_client.py` (list_nodes / run / transfer / ping). Binary Go static, cross-compile dễ (`CGO_ENABLED=0 GOOS=linux GOARCH=arm64`) — không cần cài Python trên node. Trạng thái: Phase 1 hoàn tất, E2E verified; một số đường (relay fallback qua NAT, key validation với hub) ghi rõ "needs real infra to exercise". (https://github.com/aristoteleo/PantheonOS/tree/main/fleet)

- Orchestration: có — task dispatch qua NATS subject, heartbeat/registry, health_check đếm worker online, retry/timeout/max_concurrent trên worker config. (docs distributed.html L106–L145, L332–L362)

## Yêu cầu triển khai

- Python ≥3.10, `pip install pantheon-agents` (hoặc Docker `nanguage/pantheon-agents:latest`). Đáng chú ý: package **đóng gói sẵn `nats-server-bin`** (không cần cài NATS riêng); phụ thuộc OpenAI-compatible qua `openai>=2.0.0`. (https://raw.githubusercontent.com/aristoteleo/PantheonOS/main/pyproject.toml L50–L152)
- LLM: setup wizard hỗ trợ **"OpenAI-Compatible Fallback" (LLM_API_BASE + LLM_API_KEY)** — xác minh trong source hiện tại tại https://github.com/aristoteleo/PantheonOS/blob/main/pantheon/repl/setup_wizard.py (L92–L98, L220–L248); package còn chứa snapshot catalog OpenRouter (`pantheon/data/*.json`, pyproject L317–L325) → dùng model free qua OpenRouter khả thi.
- Fleet: cần build 4 binary Go (`fleet`, `fleetctl`, `fleet-controller`, `fleet-relay`) + 1 NATS server bật JetStream; node chỉ cần chạy binary (~MB), rất nhẹ.
- **Cảnh báo bảo mật (từ chính README):** tháng 06/2026 sự cố "Hades" — bản `pantheon-agents` 0.6.1/0.6.2 trên PyPI bị trojan, đã gỡ; bản hiện tại ghi là sạch. Nếu từng cài 2 bản đó phải gỡ + rotate credential. (README L219–L227)

## Docs có đủ để dựng không?

Có, rất đầy đủ: trang Distributed Deployment (NATS setup, settings.json, start worker, multi-worker, HA, TLS, monitoring, docker-compose, k8s) tại https://pantheonos.readthedocs.io/en/latest/advanced/distributed.html; fleet có quickstart build + join node tại https://github.com/aristoteleo/PantheonOS/tree/main/fleet. Điểm chưa xác minh được: chưa kiểm chứng binary fleet chạy thực tế trên Termux/Android (dù Go static arm64 về lý thuyết chạy được).

## Verdict — Mạnh deploy được không? Được có điều kiện

- Distributed thật cả 2 tầng (NATS worker đa máy + Fleet runner chạy lệnh trực tiếp trên node); docs dựng rất chi tiết; OpenAI-compatible fallback → dùng được OpenRouter free; **Fleet bằng Go static nhẹ là ứng viên tốt nhất cho 2 điện thoại + VPS yếu trong 2 repo này.**
- Điều kiện: (1) repo còn Beta, docs cảnh báo "APIs may change"; (2) dính sự cố supply-chain 06/2026 — chỉ cài bản mới nhất từ PyPI, kiểm tra version, không dùng bản 0.6.1/0.6.2; (3) core hướng data science/bioinformatics, dependency nặng (crawl4ai, lancedb, jupyter, modal...) — nếu chỉ cần fleet/distributed thì nên tách dùng riêng phần fleet + NATS, tránh cài full; (4) Fleet mới Phase 1, một số đường relay qua NAT chưa được verify thực tế.

## Nguồn (URL cụ thể đã đọc)

- https://github.com/aristoteleo/PantheonOS (README: distributed architecture, Pantheon-Fleet, cảnh báo sự cố Hades 06/2026, stars 490, license BSD-2-Clause theo trang repo — lưu ý pyproject ghi MIT, chưa xác minh bản nào đúng)
- https://pantheonos.readthedocs.io/ và https://pantheonos.readthedocs.io/en/latest/advanced/distributed.html (toàn bộ kiến trúc + triển khai NATS multi-machine)
- https://github.com/aristoteleo/PantheonOS/tree/main/fleet (Pantheon-Fleet: runner, control plane, data plane libp2p, quickstart, trạng thái Phase 1)
- https://raw.githubusercontent.com/aristoteleo/PantheonOS/main/pyproject.toml (Python ≥3.10, nats-py + nats-server-bin, openai>=2.0.0, snapshot catalog OpenRouter)
- https://github.com/aristoteleo/PantheonOS/blob/main/pantheon/repl/setup_wizard.py (xác minh trong source: OpenAI-Compatible Fallback LLM_API_BASE + LLM_API_KEY)
- https://deepwiki.com/aristoteleo/pantheonos/2-getting-started (nguồn phụ: custom endpoint qua setup wizard)
