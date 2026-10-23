# subagent-fleet

- URL: https://github.com/adityak74/subagent-fleet
- Stars: 5 (tại thời điểm đọc)
- Đánh giá ngày: 23/10/2026

## Là gì (2 dòng)

Control plane bằng Python cho Claude Code / coding agent: đọc một file `fleet.yaml` liệt kê các node Ollama, rồi sinh ra config LiteLLM gateway + agent definitions, kèm dashboard giám sát health/warmup/trace theo thời gian thực. Không phải orchestrator giao task xuống máy — nó là lớp **định tuyến model theo vai trò** (planner → model nhẹ, implementer → model code nặng) trên nhiều máy chạy Ollama.

## Kiến trúc multi-device (chi tiết)

- **Topology khai báo trong `fleet.yaml`** (schema lấy từ PyPI README): `project` (tên + gateway LiteLLM 127.0.0.1:4000), `nodes` (mỗi node chỉ có `endpoint` dạng `http://IP:11434` + `tags`), `models` (mỗi model gắn 1 `node`, trường model là `ollama_model`, có `litellm_alias`, `context`, `timeout`, `max_parallel`), `agents` (vai trò: planner/implementer/reviewer/summarizer, mỗi vai trò gắn 1 model + danh sách tool). Nguồn: https://pypi.org/project/subagent-fleet/
- **Luồng chạy:** `subagent-fleet init` → sửa `fleet.yaml` → `validate` → `discover` (ping các Ollama endpoint) → `generate` sinh `litellm_config.yaml` + `.claude/agents/*.md` + `.env.subagent-fleet` → chạy `litellm --config ./litellm_config.yaml --port 4000` → Claude Code trỏ vào gateway local. Nguồn: https://pypi.org/project/subagent-fleet/
- **"Multi-device" ở đây = nhiều máy phục vụ LLM**, không phải nhiều máy thực thi task. Worker không được spawn: agent vẫn chạy ở máy controller (nơi chạy Claude Code); các node chỉ là Ollama server cung cấp model. README minh họa: laptop (planner), Mac mini 64GB (implementer), workstation — đều là máy chạy Ollama. Nguồn: https://github.com/adityak74/subagent-fleet
- **Tailscale-aware (v0.3):** flag `--tailscale` tự resolve endpoint của peer trong tailnet để routing ổn định qua LAN — khớp với mesh Tailscale của Mạnh. Nguồn: https://github.com/adityak74/subagent-fleet
- **OpenRouter có trỏ được không?** Schema `models:` bắt buộc `node:` + `ollama_model:` — không có field cho endpoint OpenAI-compatible/remote. Workaround duy nhất là sửa tay `litellm_config.yaml` sau khi `generate` (LiteLLM hỗ trợ openrouter native), nhưng lần `generate` sau sẽ ghi đè. Evals của repo dùng `OPENROUTER_API_KEY` chỉ để chấm điểm so sánh frontier, không phải để routing. Kết luận: **không hỗ trợ chính thức**. Nguồn: https://pypi.org/project/subagent-fleet/ , https://github.com/adityak74/subagent-fleet

## Yêu cầu triển khai

- Mỗi node phải chạy **Ollama** với model đã pull sẵn (vd `qwen2.5-coder:32b` cần ~20GB RAM/VRAM; model 7b Q4 ~4–5GB). Trên CPU A1 Flex 12GB chỉ chạy được model nhỏ.
- Máy controller: Python 3.13+ (badge), `pip install subagent-fleet` (có trên PyPI), LiteLLM proxy (`pip install 'litellm[proxy]'`), Claude Code + plugin của repo (`.claude-plugin`, plugin marketplace). Nguồn: https://github.com/adityak74/subagent-fleet , https://pypi.org/project/subagent-fleet/
- RAM control plane nhẹ (CLI + LiteLLM proxy + dashboard SSE ở :8080), chạy được trên Botkeep 2GB hoặc A1 Flex; VPS Oracle 1GB chật.
- License: MIT. Nguồn: https://github.com/adityak74/subagent-fleet

## Docs có đủ để dựng không?

Cơ bản đủ: README có Quickstart → Configuration (schema fleet.yaml đầy đủ) → Commands → Ollama Worker Setup → Security; thêm `docs/` + site mkdocs + 482 tests/evals. Điểm trừ: repo nhỏ (5 stars, 45 commits, tạo 15/06/2026), cộng đồng gần như không có, rủi ro bỏ dở cao. Nguồn: https://github.com/adityak74/subagent-fleet , https://pypi.org/project/subagent-fleet/

## Verdict — Mạnh deploy được không?

**Được có điều kiện.** Deploy kỹ thuật được (MIT, pip install, Tailscale-aware khớp mesh của Mạnh), nhưng: (1) fleet.yaml **chỉ chấp nhận Ollama endpoints** — OpenRouter free của Mạnh không gắn vào được qua config chính thức; (2) nó **không phải orchestrator giao việc** — thiết bị của Mạnh (S8+/S20FE/A1) chỉ đóng vai "máy phục vụ model", mà điện thoại chạy Ollama + model code là không thực tế, còn A1 CPU chỉ gánh model nhỏ; (3) repo 5 stars, một tác giả, nguy cơ maintain thấp. Hợp nhất nếu Mạnh muốn thử pattern "gateway LiteLLM + định tuyến theo vai trò" trên A1 Flex, không hợp nếu cần điều phối task đa thiết bị.

## Nguồn (URL cụ thể đã đọc)

- https://github.com/adityak74/subagent-fleet (README, kiến trúc, features, stars/license)
- https://pypi.org/project/subagent-fleet/ (schema fleet.yaml, quickstart, luồng generate)
