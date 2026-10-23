# Research: Multi-Device Agent Repos

*Săn repo GitHub chạy điều phối agent ĐA THIẾT BỊ/đa node (orchestrator + agent trên nhiều máy) — đánh giá xem triển khai được không. Không giới hạn cấu hình thấp.*

Repo anh em: [agent-fleet-research](https://github.com/Manhnt-adm/agent-fleet-research) (nghiên cứu kiến trúc sâu + bộ sườn MVP). Repo này chỉ làm một việc: tìm repo người ta đã làm multi-device, đọc docs, verdict triển khai được hay không.

## Kết quả nổi bật (23/10/2026)

**Deploy được:**
- **rook** (Bake-Ware/rook) — điều khiển mọi máy từ một chỗ, E2E ChaCha20-Poly1305, worker Python nhẹ, **APK Android không cần root** (tap/swipe qua AccessibilityService)
- **lamda** (firerpa/lamda, 8.6k★) — lái Android bằng ngôn ngữ tự nhiên qua OpenAI-compatible API → OpenRouter free được. Lưu ý: mặc định không auth (chỉ dùng trong Tailscale)
- **hatchet** (hatchet-dev/hatchet, 8.1k★) — orchestration engine Go, worker gRPC dial-out; cần Postgres → hợp máy 12GB (A1 Flex)

Chi tiết: `top-verdicts.md` và `assessed/<tên>.md`.

## Cấu trúc

- `candidates.md` — danh sách ứng viên theo đợt quét (discovered/assessing/assessed/dropped/noted)
- `top-verdicts.md` — tổng hợp verdict từng đợt
- `assessed/` — mỗi repo 1 file: là gì → kiến trúc multi-device → yêu cầu triển khai → verdict

## Quy tắc

- Mọi khẳng định kèm URL repo/docs cụ thể; không tìm thấy ghi "chưa xác minh được", không bịa.
- Chỉ đọc, không cài đặt/chạy code lạ.
