# Mesh-LLM
- URL: https://github.com/Mesh-LLM/mesh-llm
- Stars: 3546 (tại thời điểm đọc; repo tạo 11/02/2026, 2.865 commit, license Apache-2.0)
- Đánh giá ngày: 23/10/2026

## Là gì (2 dòng)
Mesh-LLM là **mạng inference LLM phân tán (P2P)**: gom GPU/RAM của nhiều máy thành một pool chung và expose ra một API OpenAI-compatible tại `http://localhost:9337/v1` cho chat và agents. Nói ngắn gọn: nhiều máy cùng "cày" chung một/bộ model, client/agent chỉ gọi một endpoint thống nhất.

## Kiến trúc multi-device (chi tiết: các node giao tiếp bằng gì, ai điều phối ai, có chạy agent trên nhiều máy được không)
- **Giao tiếp:** node-to-node qua **QUIC mã hóa end-to-end** (kể cả prompt, response, và split-model activations); dùng **iroh relays** làm trung gian forward gói mã hóa khi không nối trực tiếp được — relay không đọc được payload. Nguồn: https://github.com/Mesh-LLM/mesh-llm (README, mục "How the mesh works") và https://raw.githubusercontent.com/Mesh-LLM/mesh-llm/main/mesh/docs/MESHES.md (mục "Create an unpublished mesh and join by token").
- **Discovery:** mesh public quảng bá qua **Nostr discovery**; mesh private/unpublished join bằng **invite token** (token là "connectivity material", KHÔNG phải credential bearer — docs cảnh báo rõ "unpublished is not the same as access-controlled", phải cấu hình owner trust-policy riêng). Có chế độ mDNS chỉ-LAN. Nguồn: MESHES.md các mục "Join tokens, discovery, and admission", "Publish your own mesh".
- **Ai điều phối ai:** mesh router định tuyến request theo field `model` tới node có thể serve model đó; với model lớn quá một máy, **coordinator** (Skippy stage splits) lập kế hoạch chia layer-range liên tục cho nhiều node, start stage downstream trước rồi mới publish stage-0 route. Node có thể vừa serve vừa làm API-only client, hoặc client thuần (`mesh-llm client`). Nguồn: README mục "How the mesh works"; MESHES.md.
- **Agent thì sao — điểm then chốt:** agent KHÔNG chạy phân tán trên nhiều máy. Mesh-LLM chỉ làm **backend LLM dùng chung**: các launcher `mesh-llm goose / claude / opencode / pi` trỏ agent tool về endpoint mesh để agent (chạy local, trên máy có agent) gọi model qua API OpenAI-compatible. Tính năng "Mixture-of-Agents" (`model: "mesh"`) là **fan-out song song một prompt tới mọi model trong mesh rồi arbitrate** — tức nhiều model cùng trả lời một câu hỏi, không phải nhiều agent chia việc trên nhiều máy. Plugin **blackboard** chỉ là bảng tin chia sẻ status/câu hỏi giữa các peer (có MCP endpoint tại `:3131/mcp`), không phải orchestration task. Nguồn: https://raw.githubusercontent.com/Mesh-LLM/mesh-llm/main/mesh/docs/AGENTS.md (toàn file).
- **Kết luận kiến trúc:** đây là **chia sẻ compute/inference**, không phải **chạy agent phân tán đa thiết bị**. Mạnh muốn agent "chân tay" chạy trên nhiều máy — repo này không làm việc đó.

## Yêu cầu triển khai (cần cài gì, phụ thuộc nặng nhẹ, phần cứng yêu cầu)
- **Cài:** binary prebuilt (installer `install.sh` / `install.ps1`, Homebrew cho Apple Silicon, package Ubuntu/Arch, OCI image) hoặc build từ source bằng `just` (cần `just`, `cmake`, Rust, `sccache`, linker nhanh). Nguồn: README mục "Quick start", "Install and build notes"; https://raw.githubusercontent.com/Mesh-LLM/mesh-llm/main/install.md.
- **OS/arch được hỗ trợ chính thức:** macOS (Apple Silicon), Linux x86_64, **Linux aarch64** (CPU và CUDA), Windows — KHÔNG có bản Android/Termux chính thức. Nguồn: README "Install and build notes"; install.md ("Unix installer supports Apple Silicon macOS, Linux x86_64, and Linux aarch64").
- **Phần cứng:** bản chất là pool GPU/VRAM; node nhỏ (1–8GB model-fit capacity) được offer model default nhỏ (`meshllm/laya-multilingual-F16-GGUF`) — model thực tế kiểu `Qwen3-8B-Q4_K_M` nặng ~5GB. Không có máy nào trong infra của Mạnh có GPU (Oracle free tier CPU-only; điện thoại và Botkeep CPU). Chạy CPU được về mặt kỹ thuật nhưng inference chậm và ăn RAM. Nguồn: MESHES.md mục "Try the public mesh"; README.
- **Mạng:** QUIC direct hoặc qua iroh relay — có thể chạy trên Tailscale về mặt nguyên tắc, nhưng docs không đề cập Tailscale; chưa xác minh được.

## Docs có đủ để dựng không? (đánh giá trung thực độ đầy đủ docs)
Docs **rất đầy đủ và trung thực** — điểm cộng lớn. Có: README hướng quick start, `mesh/docs/` gồm MESHES.md (private/public mesh, join token, admission), AGENTS.md (tích hợp Goose/Claude Code/OpenCode/Pi + blackboard), CLI.md, USAGE.md, config reference, SDK (Rust/Node/Swift/Kotlin-Android), SKIPPY_SPLITS.md, `install.md` là runbook từng bước cho assistant dựng mesh private (hỏi Q&A, inventory từng máy, verify sau mỗi bước). Đáng chú ý: docs **không thổi phồng** — nói thẳng token join không phải auth, attestation chỉ chứng minh binary chứ không chứng minh runtime không bị sửa, MoA còn experimental. Chưa xác minh được: mức RAM/CPU tối thiểu chính xác cho node client-only (fetch USAGE.md bị rate-limit 429 khi đọc).

## Verdict — Mạnh deploy được không?
**Loại** — 3 lý do gắn với hạ tầng của Mạnh:
1. **Sai bài toán của track:** track cần repo chạy **agent phân tán đa thiết bị**; Mesh-LLM chỉ là **chia sẻ compute để inference** — agent vẫn chạy local và chỉ mượn model của mesh. Không có orchestration task, không phân phối việc cho nhiều máy.
2. **Hạ tầng không có GPU, giá trị thay thế ≈ 0:** toàn bộ giá trị của repo là pool GPU/VRAM; Mạnh không có GPU nào (Oracle free CPU, điện thoại CPU, Botkeep 2GB) mà đã có **model free qua OpenRouter** — dựng mesh để serve Qwen 8B bằng CPU 12GB A1 vừa chậm vừa không rẻ hơn được nữa.
3. **Điện thoại ngoài ma trận hỗ trợ:** installer chính thức không có bản Android/Termux; node client-only trên S8+/S20FE chưa xác minh được có chạy được không.

**Trường hợp duy nhất có thể xem lại:** sau này nếu A1 Flex có GPU hoặc Mạnh muốn một backend LLM self-hosted hoàn toàn không phụ thuộc OpenRouter — khi đó Mesh-LLM là ứng cử viên mạnh (docs tốt, private mesh bằng invite token chạy được sau Tailscale). Hiện tại: không.

## Nguồn (liệt kê URL cụ thể đã đọc)
- https://github.com/Mesh-LLM/mesh-llm (README: tổng quan, "How the mesh works", "Mixture-of-Agents", "Install and build notes"; số sao 3546, commit, license)
- https://raw.githubusercontent.com/Mesh-LLM/mesh-llm/main/mesh/docs/MESHES.md (join token/discovery/admission, unpublished mesh, private mesh, API-only client, mDNS LAN)
- https://raw.githubusercontent.com/Mesh-LLM/mesh-llm/main/mesh/docs/AGENTS.md (launcher goose/claude/opencode/pi, blackboard + MCP endpoint)
- https://raw.githubusercontent.com/Mesh-LLM/mesh-llm/main/install.md (runbook setup mesh private, OS/arch hỗ trợ, survey phần cứng)
- https://github.com/Mesh-LLM/mesh-llm/tree/main/mesh/docs (mục lục docs: CLI.md, USAGE.md, SDK, SKIPPY_SPLITS.md, EXO_COMPARISON.md, design/)
