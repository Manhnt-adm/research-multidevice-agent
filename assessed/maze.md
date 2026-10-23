# Maze

- URL: https://github.com/maze-agent/Maze
- Stars: 608 (tại thời điểm đọc)
- Đánh giá ngày: 23/10/2026

## Là gì (2 dòng)

Framework Python biến agent program thành workflow phân tán có thể quan sát được: client (SDK Python / LangGraph / Workbench UI) submit DAG `maze.workflow/v1` lên Maze Core, Core lên lịch chạy từng task trên cluster. (https://github.com/maze-agent/Maze — README L124–L145)

## Kiến trúc multi-device (chi tiết)

- **Distributed là đa máy vật lý thật, không phải parallel trong 1 process.** Kiến trúc: client → POST /workflows/submit → Maze Core (FastAPI, run state/events/logs/artifacts) → Scheduler + **Ray** → head node + worker nodes. (README L146–L191)
- Worker chạy trên máy khác: máy head `maze start --head --port 8000 [--ray-head-port 6379]`, máy worker `maze start --worker --addr HEAD_IP:8000`; worker tự đăng ký CPU/GPU/RAM với head qua `/start_worker`. (README L259–L289; xác nhận thêm tại https://deepwiki.com/maze-agent/Maze/2-getting-started — "Deployment Models / Distributed Mode")
- Scheduling riêng 2 tầng: FCFS hoặc HACS (paper-aligned) sắp xếp task vào 3 queue gpu/cpu/io; node placement strategy chọn node đủ tài nguyên. (README L291–L307; docs https://maze-doc-new.readthedocs.io/en/latest/overview.html#what-is-maze — mục "Distributed Deployment")
- Orchestration đầy đủ: run state bền vững qua restart, retry/timeout/cancel/deadline, content-addressed artifact store (`maze://artifacts/sha256/...`) để truyền file giữa worker khi không có shared storage, worker re-register sau khi head/Ray restart. (README L132–L142, L457–L479; news 2026-07 L200–L202)
- Model execution: task khai báo `model_anchor` để scheduler tự deploy vLLM/Transformers instance khi node có checkpoint; **workflow cũng có thể tiêu thụ OpenAI-compatible endpoint được inject** — tức dùng được OpenRouter thay vì model local. (README L480–L520)

## Yêu cầu triển khai

- Python ≥3.10, <3.14; `pip install maze-agent` (PyPI, MIT). Phụ thuộc chính: `ray==2.50.1` (pin cứng), fastapi, uvicorn, zmq, networkx, cloudpickle, `openai>=2.15.0`, transformers/accelerate (model local, optional), vLLM chỉ là extra `inference-vllm`. (https://raw.githubusercontent.com/maze-agent/Maze/main/pyproject.toml L84–L139)
- Mỗi node cần cài Python + package; không cần Docker. Không yêu cầu GPU (queue gpu chỉ dùng khi task khai báo).
- Task là Python function được cloudpickle serialize gửi sang worker — worker phải có cùng package/môi trường Python.

## Docs có đủ để dựng không?

Có. README có quick start head/worker đầy đủ lệnh; docs readthedocs (https://maze-doc-new.readthedocs.io/) có mục Overview/Installation/Client Guides; deepwiki (nguồn phụ) mô tả chi tiết flow đăng ký worker. Điểm chưa xác minh được: docs chưa nói rõ Ray chạy ổn định trên ARM (A1 Flex) hay Android/Termux — chỉ nói OS independent.

## Verdict — Mạnh deploy được không? Được có điều kiện

- Distributed thật qua Ray (head + worker đa máy), docs dựng đủ, model gọi qua OpenAI-compatible endpoint nên dùng được OpenRouter free.
- Điều kiện: (1) đặt head trên A1 Flex 12GB (Ray + FastAPI + scheduler nặng — không đặt head lên VPS Oracle 1GB); (2) worker Python/Ray khó chạy trên 2 điện thoại Termux (Ray không thiết kế cho Android) → điện thoại không tham gia được cluster này, chỉ A1/VPS/Botkeep làm node; (3) task phải là Python function, không phải "agent tự do chạy shell trên máy" như fleet của PantheonOS.

## Nguồn (URL cụ thể đã đọc)

- https://github.com/maze-agent/Maze (README: kiến trúc, quick start head/worker, scheduling, artifacts, model execution; stars 608; license MIT)
- https://maze-doc-new.readthedocs.io/ và https://maze-doc-new.readthedocs.io/en/latest/overview.html#what-is-maze (mục "Distributed Deployment")
- https://raw.githubusercontent.com/maze-agent/Maze/main/pyproject.toml (Python ≥3.10,<3.14; ray==2.50.1 pin; openai>=2.15.0; vLLM optional)
- https://deepwiki.com/maze-agent/Maze/2-getting-started (nguồn phụ, xác nhận deployment model standalone vs distributed)
- https://pypi.org/project/maze-agent/ (nguồn phụ, xác nhận quick start multi-machine)
