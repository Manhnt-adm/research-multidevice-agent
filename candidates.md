# Candidates — Săn repo multi-device (đợt quét 23/10/2026)

Track nghiên cứu riêng cho Mạnh: tìm repo GitHub chạy điều phối agent ĐA THIẾT BỊ/đa node
(orchestrator + agent/worker trên nhiều máy). KHÔNG giới hạn cấu hình thấp.
Loại trừ (đã nghiên cứu kỹ trước đó): Hermes, OpenClaw, Orca (stablyai/orca), Herdr,
Paperclip, AutoGen, CrewAI, LangGraph, OpenHands, cortex-hub (lktiep).

TUYỆT ĐỐI không đẩy gì lên repo `agent-fleet-research` (Antigravity + Claude đang làm việc ở đó).

Trạng thái: discovered = mới phát hiện | assessing = đang đọc docs đánh giá |
assessed = đã có file assessed/<tên>.md | dropped = loại sau đánh giá | noted = ông lớn ngành (ghi chú bối cảnh, không deep-dive multi-device)

## Nhóm A — Sát use-case nhất (hub + worker đa máy) → ưu tiên deep-read đợt 2

| # | Repo | Stars | Mô tả 1 dòng | Nguồn | Trạng thái |
|---|------|-------|--------------|-------|------------|
| A1 | [Cordane](https://github.com/cordane/cordane) | 3 | Control plane cho AI coding agents (repo chỉ docs, software proprietary) — cần subscription AI trả phí, free tier 1 worker | niche query | dropped |
| A2 | [rook](https://github.com/Bake-Ware/rook) | 4 | Điều khiển mọi máy từ một chỗ (web/TUI/MCP), E2E ChaCha20-Poly1305; worker Python nhẹ, có APK Android không-root (tap/swipe qua AccessibilityService) | niche query | assessed |
| A3 | [ai-maestro](https://github.com/23blocks-OS/ai-maestro) | 834 | Dashboard điều phối 80+ agent; peer mesh no-central-server, worker headless ~100MB — nhưng thiết kế cho Claude Code/Codex trả phí | web/awesome | assessed |
| A4 | [AgentMesh](https://github.com/johannschiffer/agentmesh-mcp) | ? | Task board MCP: claim nguyên tử SQLite WAL, priority queue; bản open stdio-only (1 máy), zombie reclaimer là Pro $14.99 — đáng cắp ý tưởng cho MVP | web/dev.to | assessed |
| A5 | [Muxin](https://github.com/LumaryLabsLLC/muxin) | 362 | Desktop app Rust (Apache 2.0) gom nhiều máy qua SSH/Tailscale; session persistent, MCP 89 tools — nhưng agent CLI cần subscription trả phí | niche query | assessed |
| A6 | [figaro](https://github.com/byt3bl33d3r/figaro) | 155 | Fleet Claude Computer Use qua bus NATS — khóa cứng Claude trả phí, mỗi worker = cả desktop Linux + Chromium (nặng); SECURITY.md chỉ đích danh Tailscale làm lớp mạng | github search | dropped |
| A7 | [lamda](https://github.com/firerpa/lamda) | 8647 | Android Full-Stack Device Control: server trên máy (root/Shizuku), lệnh `agent` lái máy bằng ngôn ngữ tự nhiên qua OpenAI-compatible API — ứng viên mạnh cho vai "tay" 2 điện thoại | github search | assessed |
| A8 | [phleet](https://github.com/anurmatov/phleet) | 21 | .NET 10, ~8GB RAM (MySQL/Qdrant/Temporal/MinIO/RabbitMQ), cần sub AI trả phí; chạy 1 host Docker — KHÔNG multi-device | web/dev.to | dropped |
| A9 | [MagiC](https://github.com/kienbui1995/magic) | 0 | "Kubernetes for AI agents" (tác giả Việt): server Go + SQLite, worker HTTP, Ollama local — deploy được nhưng 0 star/1 tác giả/1 tháng tuổi, chỉ học/prototype | web/awesome | assessed |
| A10 | [hatchet](https://github.com/hatchet-dev/hatchet) | 8169 | Orchestration engine Go (MIT): worker long-lived gRPC về engine, Postgres bắt buộc — control plane đặt lên A1 Flex 12GB, worker nhẹ chạy VPS nhỏ/S8+ qua Tailscale | github+niche | assessed |

## Nhóm B — Đáng đánh giá tiếp (đợt 2 batch 2 hoặc đợt 3)

| # | Repo | Stars | Mô tả 1 dòng | Nguồn | Trạng thái |
|---|------|-------|--------------|-------|------------|
| B1 | [agentscope-java](https://github.com/agentscope-ai/agentscope-java) | 6091 | "Build distributed, production-grade, long-running agents" (Java) | github search | dropped |
| B2 | [thurbox](https://github.com/Thurbeen/thurbox) | ? | TUI Rust điều phối coding agent song song; remote host qua SSH trong hosts.toml | web/awesome | assessed |
| B3 | [termany](https://github.com/thinkany-ai/termany) | ? | Terminal nhiều coding agent một cửa sổ, per-pane SSH tới remote host | web/awesome | dropped |
| B4 | [mcloud](https://github.com/zebrunner/mcloud) | 204 | Mobile farm ecosystem cho Android/iOS (tập trung nhiều thiết bị) | github search | dropped |
| B5 | [subagent-fleet](https://github.com/adityak74/subagent-fleet) | ? | CLI điều phối fleet qua fleet.yaml liệt kê Ollama node endpoints | web/pypi | assessed |
| B6 | [SwarmForge](https://github.com/GavinGeizer/swarmforge-oss) | ? | Orchestrator tự host: coordinator 1 máy, mỗi worker 1 VM riêng, giao việc qua MCP | web/dev.to | dropped |
| B7 | [Mesh-LLM](https://github.com/Mesh-LLM/mesh-llm) | 3546 | Mesh chia sẻ compute (private/public) để cấp sức cho agents — P2P phân tán | github+topstar | dropped |
| B8 | [yomo](https://github.com/yomorun/yomo) | 1929 | Serverless AI Agent Framework trên hạ tầng edge phân tán địa lý | github+topstar | dropped |
| B9 | [golem](https://github.com/golemcloud/golem) | 1516 | Agent-native platform cho AI agents + distributed apps không mất state | github search | dropped |
| B10 | [SmythOS/sre](https://github.com/SmythOS/sre) | 1303 | Cloud-native runtime cho agentic AI: local, cloud, edge | github search | assessed |
| B11 | [IoA](https://github.com/OpenBMB/IoA) | 836 | Framework cho collaborative AI agents phân tán, kết nối kiểu internet | github search | dropped |
| B12 | [Ruflo](https://github.com/ruvnet/ruflo) | ? | Meta-harness chạy swarm tới 100 coding agent; federation cross-machine qua mTLS + Ed25519 | niche/dev.to | assessed |
| B13 | [MCP-Edge](https://github.com/jemsbhai/mcp-edge) | ? | Gateway mở rộng MCP xuống edge/IoT; MCP-Lite ~512KB RAM trên thiết bị | niche query | dropped |
| B14 | [Maze](https://github.com/maze-agent/Maze) | 608 | Distributed framework cho LLM agents | github search | assessed |
| B15 | [PantheonOS](https://github.com/aristoteleo/PantheonOS) | 490 | Distributed agent framework cho data science | github search | assessed |
| B16 | [Octop](https://github.com/TencentCloud/Octop) | ? | AI assistant tự host Tencent: multi-user/agent, offload cloud↔edge, ACP runners | web | dropped |
| B17 | [AutoBot-AI](https://github.com/mrveiss/AutoBot-AI) | ? | Nền tảng AI tự host + quản lý fleet qua Ansible (rolling/canary/blue-green) | niche/dev.to | dropped |
| B18 | [RobotFleet](https://github.com/therohangupta/robot-fleet) | ? | Framework lập kế hoạch & scheduling tập trung cho fleet robot dùng LLM | niche/arXiv | assessed |
| B19 | [Open-RMF](https://github.com/open-rmf/rmf_demos) | ? | Middleware cho nhiều fleet robot khác hãng: phân task, giải xung đột | niche query | assessed |
| B20 | [zerus](https://github.com/ufna/zerus) | 14 | ADE quản lý fleet AI coding agents "across your machines", sync project qua P2P | github search | assessed |
| B21 | [code-buddy](https://github.com/phuetz/code-buddy) | 48 | Local-first AI coding agent: 220+ tools, peer-to-peer fleet | github search | assessed |
| B22 | [symphony](https://github.com/GradientHQ/symphony) | 48 | Decentralized multi-agent framework trên edge devices không đồng nhất | github search | dropped |
| B23 | [mcp-mesh](https://github.com/dhyansraj/mcp-mesh) | 42 | Distributed AI agent framework K8s-native, auto-failover, MCP+A2A+REST | github search | assessed |
| B24 | [cycls](https://github.com/Cycls/cycls) | 27 | SDK chuyên "distributing AI agents" ra nhiều node | github search | dropped |
| B25 | [oryxos](https://github.com/oryx-labs/oryxos) | 194 | Tự nhận "Distributed AI Agent OS" | github search | assessed |
| B26 | [hyperspaceai/agi](https://github.com/hyperspaceai/agi) | 2072 | Distributed AGI: hàng nghìn agent P2P gossip, join từ browser/CLI | github search | dropped |
| B27 | [skillhub](https://github.com/iflytek/skillhub) | ? | Skill registry tự host cho fleet agent (hạ tầng phụ trợ, không phải orchestrator) | web/dev.to | assessed |
| B28 | [astron-agent](https://github.com/iflytek/astron-agent) | ? | Workflow orchestration trên skillhub | web/dev.to | dropped |
| B29 | [OpenSTF](https://github.com/openstf/stf) | ? | Web app quản lý farm thiết bị Android vật lý (không AI — giữ vì pattern hub+thiết bị) | niche query | assessed |
| B30 | [grlx](https://github.com/gogrlx/grlx) | 226 | Fleet config management Go qua NATS (không phải LLM agent — pattern tham khảo) | github search | assessed |

## Nhóm C — Ông lớn ngành (ghi chú bối cảnh, không deep-dive multi-device)

| Repo | Stars | Vì sao đáng biết |
|------|-------|------------------|
| [dify](https://github.com/langgenius/dify) | 159299 | Nền tảng agentic workflow nhiều sao nhất (~159k); self-hosted được |
| [deer-flow](https://github.com/bytedance/deer-flow) | 84299 | SuperAgent harness của ByteDance: subagents + message gateway |
| [MetaGPT](https://github.com/FoundationAgents/MetaGPT) | 70974 | Framework multi-agent kinh điển đời đầu, mô hình "AI software company" |
| [agentscope](https://github.com/agentscope-ai/agentscope) | 33694 | Multi-agent; bản java (6k★) nhấn mạnh distributed/long-running |
| [openai-agents-python](https://github.com/openai/openai-agents-python) | 30239 | Framework chính chủ OpenAI — pattern chuẩn ngành |
| [haystack](https://github.com/deepset-ai/haystack) | 26814 | Orchestration framework lâu đời, production-ready pipelines |
| [swarm](https://github.com/openai/swarm) | 22071 | Repo gốc pattern swarm/handoff, vô số framework copy theo |
| [adk-python](https://github.com/google/adk-python) | 21971 | ADK chính chủ Google: build/evaluate/deploy có hệ thống |
| [eliza](https://github.com/elizaOS/eliza) | 19639 | "Agentic OS" — gợi ý runtime đa agent đa host, cần đọc sâu nếu rảnh |
| [camel](https://github.com/camel-ai/camel) | 17955 | Framework multi-agent đầu tiên (2023) |
| [trigger.dev](https://github.com/triggerdotdev/trigger.dev) | 16605 | Durable AI agents/workflows — runtime phân tán phía sau |
| [agent-framework](https://github.com/microsoft/agent-framework) | 14282 | Framework chính thống Microsoft (Python + .NET) |
| [swarms](https://github.com/kyegomez/swarms) | 7287 | Enterprise multi-agent orchestration |

## Ghi chú quét
- 4 trinh sát (GitHub search ×10 query, web/awesome-lists, niche 7 query, top-starred ×7 query), 23/10/2026.
- Số sao ghi tại thời điểm quét; "?" = chưa lấy số (lấy khi deep-read).
- Đánh giá khả năng triển khai cho Mạnh nằm ở `assessed/<tên>.md`.
