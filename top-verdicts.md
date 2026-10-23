# Top Verdicts — Săn repo multi-device (23/10/2026)

Đánh giá sâu 10 repo nhóm A (đọc docs gốc, có URL nguồn trong từng file `assessed/`).
Quy tắc: mọi verdict đều trung thực với hạ tầng của Mạnh (VPS 1GB, S8+/S20FE qua Termux,
Tailscale mesh, KHÔNG subscription AI trả phí, A1 Flex 12GB sắp dựng).

## ✅ ĐƯỢC — deploy được

### 1. rook (Bake-Ware/rook) — phù hợp nhất track
- Fleet management tự host: worker dial-out về hub qua relay mù (Rust), không mở port; E2E ChaCha20-Poly1305.
- Worker pure-Python nhẹ; **APK Android native không cần root** (tap/swipe qua AccessibilityService — đúng thứ skill phone-control cần); MCP server tự host; docs rất đầy đủ.
- Trừ: repo mới (3/2026), 4 sao, single maintainer, PSK là single point of failure.
- File: `assessed/rook.md`

### 2. lamda (firerpa/lamda, 8647★, MIT) — ứng viên mạnh cho vai "tay" 2 điện thoại
- Server chạy trên máy Android (root hoặc Shizuku), client Python trên PC; remote desktop WebRTC/H.264, 160+ API.
- Lệnh `agent` lái máy bằng ngôn ngữ tự nhiên qua **bất kỳ OpenAI-compatible API nào** → trỏ về OpenRouter free khả thi (cần xác minh model free có tool calls).
- S8+ (root) full năng lực; S20FE qua Shizuku (giới hạn tính năng đặc quyền).
- **Cảnh báo bảo mật**: mặc định KHÔNG có authentication — chỉ dùng trong mạng tin cậy (Tailscale mesh của Mạnh phù hợp).
- Caveat: Pigeon hub còn non (46★); rủi ro khóa nick nếu auto thao tác hàng loạt trên FB/TikTok.
- File: `assessed/lamda.md`

### 3. hatchet (hatchet-dev/hatchet, 8169★, MIT) — control plane đặt lên A1 Flex
- Engine Go: worker long-running phân tán, chủ động mở gRPC stream hai chiều về engine (Register → Heartbeat → Listen, tự reconnect; mất heartbeat → việc giao sang worker khác).
- Worker dial-out → chạy trên S8+/Termux qua Tailscale khả thi; worker nhẹ (Python SDK + token).
- **Postgres bắt buộc** → KHÔNG đặt control plane lên VPS 1GB; đặt lên A1 Flex 12GB thì vừa đẹp.
- Docs đầy đủ (docs.hatchet.run).
- File: `assessed/hatchet.md`

## ⚠️ ĐƯỢC CÓ ĐIỀU KIỆN

### 4. Muxin (LumaryLabsLLC/muxin, 362★, Apache 2.0) — "khó vừa"
- Desktop Rust gom nhiều máy qua SSH/Tailscale; session persistent kiểu tmux; MCP 89 tools; có bản headless cho VPS.
- Nhưng agent CLI thông minh (Claude Code/Codex) đòi subscription trả phí; đây là hub điều khiển, không phải conductor 24/7; dự án mới 2 tuần tuổi.
- File: `assessed/muxin.md`

### 5. ai-maestro (23blocks-OS/ai-maestro, 834★, MIT)
- Peer mesh "no central server" đã xác minh (discovery qua IP Tailscale, sync eventually consistent); messaging ký Ed25519; worker headless ~100MB; docs rất đầy đủ.
- Nhưng thiết kế cho Claude Code/Codex/Grok trả phí; build cần 4GB RAM.
- File: `assessed/ai-maestro.md`

### 6. MagiC (kienbui1995/magic, 0★) — chỉ học/prototype
- Tác giả Việt; server Go + SQLite chạy ngon trên VPS 1GB; worker HTTP protocol mở; approval gate + cost controller + circuit breaker; Ollama local (không tốn tiền AI).
- Nhưng 0 stars / 1 tác giả / 1 tháng tuổi — chưa ai kiểm chứng production.
- File: `assessed/magic.md`

## ❌ LOẠI

| Repo | Lý do |
|------|-------|
| Cordane | Software proprietary (repo chỉ docs); cần sub AI trả phí; free tier 1 worker |
| phleet | ~8GB RAM, sub AI trả phí, chạy 1 host Docker duy nhất — không multi-device thật |
| figaro | Khóa cứng Claude trả phí; mỗi worker = cả desktop Linux + Chromium |

## 📐 Ý tưởng "đánh cắp" cho MVP/conductor mới

| Nguồn | Ý tưởng |
|-------|---------|
| AgentMesh | Priority queue (`ORDER BY priority DESC, created_at ASC`); cột `last_heartbeat`/`lease_timeout_sec` + `reclaim_zombies()` cho watchdog tick 60s |
| Cordane | Worker dial-out + yamux multiplexing trên 1 WebSocket (NAT-traversal) |
| Muxin | SSH-first rồi đẩy helper nhỏ lên máy; QUIC direct pairing |
| ai-maestro | Ed25519-signed agent messaging; manual peer add qua IP Tailscale |
| rook | Relay mù + domain separation key (thay PSK bằng per-device key) |
| figaro | SECURITY.md chỉ đích danh Tailscale/Headscale làm lớp mạng — xác nhận hướng đi |
| lamda | Lệnh `agent` trỏ OpenAI-compatible API về OpenRouter free cho điều khiển điện thoại |

## Bức tranh kiến trúc gợi ý từ track này (để đối chiếu với blueprint)

- **Control plane**: hatchet trên A1 Flex 12GB (nếu chấp nhận Postgres) HOẶC conductor Python+SQLite tự viết (nhẹ, đã có MVP).
- **Tầng điều khiển máy**: rook (E2E, đa nền tảng, Android no-root) đáng PoC nhất.
- **Tay vật lý (điện thoại)**: lamda — server trên máy + lệnh `agent` qua OpenRouter free.
- Cả 3 đều chạy sau Tailscale, không mở port ra internet.

## Còn lại
- Nhóm B (30 repo): discovered — đợt 3 đào tiếp nếu cần (ưu tiên: agentscope-java, Mesh-LLM, yomo, golem, thurbox/termany, mcloud, zerus).
- Nhóm C (13 ông lớn): noted — bối cảnh ngành.

---

# WAVE 3 — Đánh giá 30 repo còn lại (23/10/2026)

17 worker song song, đọc docs gốc từng repo, verdict trung thực với hạ tầng của Mạnh.
Không repo nào đạt mức "Được" tuyệt đối như rook/lamda/hatchet ở đợt 2 — nhưng có
vài ứng viên "Được có điều kiện" đáng chú ý và nhiều pattern đáng cắp.

## ⭐ Đáng chú ý nhất đợt 3 (Được có điều kiện)

### 1. PantheonOS / Pantheon-Fleet — ứng viên nhẹ nhất cho điện thoại + VPS yếu
- Subsystem **Pantheon-Fleet viết bằng Go**: binary static vài MB, agent chạy shell/python trực tiếp trên từng node, truyền file node↔node qua **libp2p không cần VPN**.
- **CẢNH BÁO BẢO MẬT**: repo từng dính trojan PyPI **"Hades"** ở bản 0.6.1/0.6.2 (06/2026) — chỉ cài bản mới nhất, kiểm tra checksum.
- Core nặng data-science nên chỉ tách dùng phần fleet; repo còn Beta.
- File: `assessed/pantheonos.md`

### 2. thurbox — TUI+SSH duy nhất có orchestration programmatic thật
- `thurbox-cli` headless: tạo/điều khiển/chụp/đóng session bằng script, mailbox giữa các session, extension Fleet với task queue → dispatch worker song song → thu `result.md`; remote host qua SSH trong `hosts.toml`.
- Điều kiện: worker vẫn là coding-agent CLI tương tác, không thay thế được conductor REST; repo mới (02/2026, 112★).
- File: `assessed/thurbox.md`

### 3. Maze — đa máy vật lý thật qua Ray
- Head + worker đăng ký tài nguyên qua Ray, scheduler riêng 3 queue gpu/cpu/io, workflow tiêu thụ OpenAI-compatible endpoint → dùng được model free qua OpenRouter.
- Điều kiện: head đặt trên A1 Flex 12GB; **điện thoại Termux không tham gia được vì Ray**.
- File: `assessed/maze.md`

### 4. Ruflo — federation cross-machine có thật
- Ed25519-signed manifest, trust ladder 5 bậc, WSS + cert pinning, ride trên Tailscale (code + ADR kiểm chứng được).
- Điều kiện: còn **alpha** (1.0.0-alpha.14); xoay quanh Claude Code/Codex, tương thích OpenRouter free chưa xác minh; cho code-agent chứ không phải điều khiển điện thoại (rook vẫn hơn ở tầng S8+/S20FE).
- File: `assessed/ruflo.md`

### 5. zerus — SSH-peer + Tailscale hợp hạ tầng, nhưng "P2P" chỉ là SSH sync metadata
- CLI Rust nhẹ, chạy được trên A1/VPS 1GB; nhưng chỉ quản lý coding agents trong danh sách hỗ trợ (**Antigravity Pro KHÔNG được support**), điện thoại chỉ làm client theo dõi, sync chỉ metadata không sync code.
- Repo mới 16 ngày tuổi, 1 maintainer — độ chín thấp.
- File: `assessed/zerus.md`

### 6. code-buddy — P2P fleet qua Tailscale chạy được Termux
- Mỗi máy chạy `buddy server` là có fleet P2P, không cần server trung tâm; nhưng là coding agent không phải orchestrator (remote tool read-only).
- **Lưu ý license: Business Source License 1.1, KHÔNG phải open source** — dùng cá nhân/nội bộ thì ổn.
- File: `assessed/code-buddy.md`

### 7. oryxos — single-node Java chạy được, có Telegram + REST + memory/audit
- One-binary Java (Spring Boot) — hợp A1 Flex 12GB hơn VPS 1GB; nhưng "distributed" multi-machine vẫn là **roadmap chưa ship**.
- File: `assessed/oryxos.md`

### 8. golem — single-node Docker OSS trên A1 được, nhưng là platform viết lại agent bằng WASM
- Không điều khiển được điện thoại (chỉ làm client gọi API); chưa xác minh image ARM64 cho A1 Flex.
- File: `assessed/golem.md`

### 9. mcp-mesh — DDDI + auto-failover rất gần triết lý conductor MVP
- Tầng K8s quá nặng (bỏ); tầng nhẹ (agent Python) cần test thực tế. Đáng lấy làm chất liệu thiết kế hơn là deploy nguyên si.
- File: `assessed/mcp-mesh.md`

### 10. smythos-sre — Node runtime nhẹ, nhưng "distributed" chỉ là multi-runtime
- Không thay thế được coordinator; connector OpenRouter chưa có trong danh sách chính thức.
- File: `assessed/smythos-sre.md`

### 11. subagent-fleet — thực ra chỉ sinh config LiteLLM định tuyến model
- `fleet.yaml` chỉ chấp nhận Ollama endpoints (không gắn OpenRouter free qua config chính thức).
- File: `assessed/subagent-fleet.md`

### 12. skillhub — skill registry thật, nhưng thừa lúc này
- Apache 2.0, chuẩn ClawHub-style; chỉ nên dựng khi số skill/agent tăng, đặt trên A1 Flex sau Tailscale.
- File: `assessed/skillhub.md`

## 📐 Pattern đáng cắp thêm (đợt 3)

| Nguồn | Pattern |
|-------|---------|
| grlx | **NATS embedded** trong server điều khiển (pub-sub + request/reply làm heartbeat/task dispatch, không cần broker riêng) + enrollment public-key mặc định TLS — đúng thứ MVP anti-fake còn thiếu |
| OpenSTF | Provider/worker model: máy cắm điện thoại làm provider, gửi **sự kiện trạng thái qua bus** thay vì poll (hub chỉ nhận event) |
| RobotFleet | Tách 3 vai **Planner (goal→DAG) / Allocator (gán task) / Executor (báo kết quả → replan)** + world-state real-time |
| hyperspaceai/agi | Ý tưởng **Pods** (mesh máy riêng + pool chung key OpenRouter + budget per-member) — tham khảo kiến trúc, không dùng tool |
| Maze | Scheduler 3 queue gpu/cpu/io + workflow tiêu thụ OpenAI-compatible endpoint |

## ❌ Loại đợt 3 (lý do chính)

| Repo | Lý do |
|------|-------|
| agentscope-java | Framework enterprise K8s/Redis/Postgres của Alibaba; "distributed" = multi-replica, không có khái niệm device/phone |
| Mesh-LLM | Mesh chia sẻ compute cho **inference**, không phải orchestration agent; Mạnh không có GPU |
| yomo | Serverless FaaS host tool cho LLM gọi — sai bài toán |
| termany | Chỉ là multiplexer TUI cho người nhìn; app desktop, không headless API |
| mcloud | Đòi cắm USB vật lý + host 32GB RAM — sai mô hình |
| SwarmForge | Worker bắt buộc là VM trả phí trên cloud Freestyle — ngược mục tiêu $0 |
| IoA | Docker + Milvus + API trả phí; demo học thuật 21 commits |
| MCP-Edge | Gateway IoT cho MCU/sensor — sai abstraction; repo stale |
| Octop | Multi-agent trong 1 process 1 máy; nặng hơn ZeroClaw, trùng chức năng |
| AutoBot-AI | SLM node ≥1.5GB, platform node 8–16GB — vượt hạ tầng |
| symphony | Đòi GPU CUDA; repo sơ khai 3 commits |
| cycls | "Distributing agents" = deploy lên cloud của họ (SaaS), không self-host |
| hyperspaceai/agi | Mạng nghiên cứu AGI công cộng — không chạy task cá nhân |
| astron-agent | Workflow doanh nghiệp iFLYTEK, stack nặng, docs tiếng Trung |
| Open-RMF | Pattern robot vật lý (ROS 2/Gazebo) — chỉ tham khảo ý tưởng |

## Kết luận wave 3

- Không tìm được "rook thứ hai": repo **deploy được ngay và hợp toàn diện** vẫn chỉ có 3 cái ở đợt 2 (rook, lamda, hatchet).
- Bổ sung đáng giá nhất: **Pantheon-Fleet** (nhẹ nhất cho phone/VPS yếu, kèm cảnh báo trojan), **thurbox** (orchestration SSH programmatic), **Maze** (Ray đa máy thật), và các pattern **NATS embedded (grlx)**, **event bus thay poll (OpenSTF)**, **Planner/Allocator/Executor (RobotFleet)** cho MVP conductor.
- Toàn bộ 40 file đánh giá nằm ở `assessed/` (10 đợt 2 + 30 đợt 3). Không đẩy gì lên GitHub theo yêu cầu.
