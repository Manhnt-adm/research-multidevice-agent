# AgentMesh
- URL: https://github.com/johannschiffer/agentmesh-mcp
- Stars: chưa lấy (truy cập trực tiếp trang repo thất bại)
- Đánh giá ngày: 23/10/2026

## Là gì (2 dòng)

AgentMesh là **MCP server Python nhẹ (task board phân tán)** để điều phối swarm agent AI: agent gọi các tool `create_task / claim_task / complete_task` qua giao thức MCP, task được lưu trong SQLite WAL với cơ chế claim nguyên tử chống tranh chấp. Tác giả phát hành bản **Lite open-source miễn phí** (atomic claim + priority queue + board status) và bản **Pro $14.99** (DAG dependencies, heartbeat & zombie lease reclaimer, shared blackboard, cost ledger, Docker) — **lưu ý: heartbeat & zombie lease reclaimer nằm trong bản Pro, KHÔNG có trong bản open-source**.

## Kiến trúc multi-device

Thiết kế theo bài giới thiệu của tác giả (dev.to, đăng 21/10/2026, cập nhật 23/10/2026):

- **MCP server chạy ở đâu:** một tiến trình Python duy nhất, giao tiếp qua **JSON-RPC stdio** (khởi động bằng `python -m agentmesh.cli run`, cấu hình trong `claude_desktop_config.json` như mọi MCP server local cho Claude Desktop/Cursor). Agent (Alpha/Beta/Gamma...) gọi tool `create_task(title, priority)` / `claim_task(agent_id)` / `complete_task(task_id, agent_id, result)` qua MCP.
- **Atomic claim implement bằng gì:** SQLite WAL + transaction `BEGIN IMMEDIATE`. Trong `claim_task`: mở `BEGIN IMMEDIATE` (SQLite đảm bảo chỉ 1 writer giữ reservation lock tại một thời điểm) → `SELECT id FROM tasks WHERE status='pending' ORDER BY priority DESC, created_at ASC LIMIT 1` → `UPDATE tasks SET status='in_progress', claimed_by=?, claimed_at=? WHERE id=? AND status='pending'` (điều kiện kép chống race) → `COMMIT`; lỗi thì `ROLLBACK`. 10 agent gọi đồng thời thì đúng 1 agent nhận được task.
- **Priority queue:** task pending được claim theo `ORDER BY priority DESC, created_at ASC` — ưu tiên cao trước, cùng ưu tiên thì FIFO.
- **Zombie lease reclaimer hoạt động ra sao (bản Pro):** mỗi claim gắn `lease_timeout_sec` (mặc định 60s); agent sống phải gọi `heartbeat(task_id, agent_id)` định kỳ để cập nhật `last_heartbeat`. Monitor `reclaim_zombies()` quét các task `in_progress`, nếu `now - last_heartbeat > lease_timeout_sec` thì reset task về `status='pending'`, `claimed_by=NULL`, `last_heartbeat=NULL` để agent khác claim lại. Bài viết đưa nguyên code mẫu của `claim_task` và `reclaim_zombies`.
- **DAG (bản Pro):** tham số `depends_on`; task có dependency chưa xong bị giữ ở trạng thái `blocked`; khi `complete_task` chạy, engine đánh giá graph và promote task đủ điều kiện lên `pending` (có kiểm tra chu trình theo mô tả).

**Điểm trung thực quan trọng:** đây là kiến trúc **"multi-agent trên cùng một máy / cùng một file SQLite"**, KHÔNG phải multi-device qua mạng. Transport là **stdio** — agent ở máy khác không thể gọi được server này. Bài viết cũng chỉ claim "coordinate... directly on your workstation or edge server". Nếu muốn chạy đa máy thật phải tự bọc thêm transport mạng (HTTP/SSE MCP server) — bản Pro có Docker nhưng không nói rõ có HTTP transport hay không (chưa xác minh được).

## Yêu cầu triển khai

- **Ngôn ngữ:** Python; tác giả claim **"zero-dependency"** (bản Lite chỉ dùng stdlib + SQLite có sẵn).
- **Dependency:** SQLite (WAL mode) — file DB local duy nhất; không cần Redis/RabbitMQ/Celery. Giao thức MCP stdio (cần MCP client như Claude Desktop/Cursor/Claude Code để dùng).
- **Nặng hay nhẹ:** rất nhẹ — một file DB + một tiến trình Python, chạy tốt trên VPS 1GB RAM hay Termux.
- **Phần cứng:** workstation/edge server bất kỳ chạy được Python 3. PyPI có gói `agentmesh-mcp 0.1.0` (26.3 kB, upload 03/06/2026) nhưng **cảnh báo:** nội dung gói PyPI này (theo trang PyPI) là một sản phẩm KHÁC — "persistent expert sessions" với CLI `agentmesh init / serve / ask`, config `agentmesh.yml`, session lưu `.agentmesh/sessions/<domain>.json` — không phải task board `create_task/claim_task` trong bài dev.to. Chưa xác minh được mối quan hệ giữa gói PyPI và repo GitHub (có thể cùng tác giả đổi hướng, hoặc trùng tên); **không nên `pip install agentmesh` kỳ vọng ra task board**.

## Docs có đủ để dựng không?

- **Chưa xác minh được README/docs trong repo** (truy cập https://github.com/johannschiffer/agentmesh-mcp thất bại — không đọc được README hay thư mục docs/).
- Bài dev.to **đủ chi tiết về mặt thiết kế** để tự implement lại: có đầy đủ code mẫu `claim_task` (BEGIN IMMEDIATE), `reclaim_zombies`, logic unblock DAG, schema ý tưởng (`tasks` với `status/claimed_by/claimed_at/last_heartbeat/lease_timeout_sec/depends_on`), cách cấu hình MCP client. Nhưng đó là **hướng dẫn viết lại**, không phải tài liệu cài đặt bản Lite (không có hướng dẫn clone/cài/requirements cho repo).
- Code thật của bản Lite (claim + priority + board status) chưa đọc được; bản Pro là sản phẩm trả phí trên Gumroad.

## Verdict — Mạnh deploy được không?

**KHÓ / KHÔNG phù hợp nguyên trạng — nhưng đáng "đánh cắp" thiết kế.** Lý do: (1) transport stdio = chỉ chạy local một máy, không giải quyết bài toán multi-device qua Tailscale của Mạnh nếu không tự viết thêm lớp HTTP; (2) đúng thứ Mạnh quan tâm nhất — heartbeat & zombie lease reclaimer — nằm trong bản **Pro trả phí $14.99**, bản open-source không có; (3) repo rất mới (bài viết 21/10/2026), chưa có dấu hiệu cộng đồng/test công khai cho bản Lite.

**Đáng "đánh cắp" cho conductor của Mạnh (đã có MVP với claim atomic 409):**
1. `BEGIN IMMEDIATE` + `UPDATE ... WHERE status='pending'` điều kiện kép — tương đương luật claim của MVP, nhưng pattern `ORDER BY priority DESC, created_at ASC` là thứ MVP chưa có → **thêm priority queue vào conductor**.
2. Thiết kế lease: thêm 2 cột `last_heartbeat` + `lease_timeout_sec` vào bảng task của MVP, agent gửi heartbeat định kỳ, watchdog tick 60s hiện tại mở rộng thành `reclaim_zombies()` reset claim quá hạn về pending — **đây chính là mảnh còn thiếu trong MVP** (MVP mới có liveness "tin một nửa", chưa có lease timeout thật).
3. Tool `board status` (trạng thái board tổng quan) — MVP nên có endpoint tương đương để quan sát thay vì query DB tay.
4. Cảnh báo từ thiết kế này: **đừng để SQLite thành điểm chia sẻ qua mạng** (SQLite qua NFS/share rất dễ corrupt) — nếu conductor chạy đa máy, mỗi máy phải nói chuyện với conductor qua HTTP API chứ không được cùng mở một file DB.

Không cần mua bản Pro: toàn bộ ý tưởng đã lộ trong bài viết miễn phí, tự implement vào MVP của Mạnh chỉ vài chục dòng.

## Nguồn
- https://dev.to/johann_schiffer/how-to-coordinate-multi-agent-ai-swarms-without-cloud-deadlocks-a-distributed-task-board-mcp-server-djb (bài giới thiệu chính: kiến trúc, code mẫu claim_task/reclaim_zombies, phân biệt Lite/Pro; đăng 21/10/2026, cập nhật 23/10/2026)
- https://pypi.org/project/agentmesh-mcp/0.1.0/ (trang PyPI: mô tả gói là "persistent expert sessions", CLI agentmesh init/serve/ask, khác với task board trong bài dev.to — bằng chứng mâu thuẫn tên gói)
- https://github.com/johannschiffer/agentmesh-mcp (repo gốc — **truy cập thất bại, chưa đọc được README/docs**, số stars chưa lấy được)
