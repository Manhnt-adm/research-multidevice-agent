# code-buddy
- URL: https://github.com/phuetz/code-buddy
- Stars: 48 (tại thời điểm đọc)
- Đánh giá ngày: 23/10/2026

## Là gì (2 dòng)
Local-first AI coding agent chạy trên terminal (TypeScript/Node, 220+ tools, 64 LLM provider trong đó 30 free/local, auto-failover). Có thể chạy dưới dạng "fleet" P2P: mỗi máy chạy `buddy server`, các peer nhìn thấy event của nhau và gọi model/tool của nhau qua network.

## Kiến trúc multi-device (ngắn gọn)
- Mỗi node chạy `buddy server`; các peer "observe each other's events" và gọi lẫn nhau qua `peer.chat`, `peer.chat-session.*`, `peer.tool.invoke` (remote tool là **read-only** qua 3 cửa: allowlist → flag `fleetSafe` → workspace root fail-closed khi chưa set). (https://github.com/phuetz/code-buddy — README mục "What Code Buddy is", đoạn "A multi-AI fleet hub")
- Không có orchestrator trung tâm cho fleet; mỗi peer là coding agent độc lập chạy tool trên máy local của mình. Federation kiến thức giữa các peer kiểu pull-only (opt-in). (https://github.com/phuetz/code-buddy — README mục "Opt-in")
- Giao tiếp peer-to-peer; repo cũng có bản desktop GUI (Electron, Cowork) nhưng không liên quan đến fleet.

## Yêu cầu triển khai (ngắn gọn)
- Node.js ≥ 20 (khuyên 22/24); cài bằng `npm i -g @phuetz/code-buddy`. (https://github.com/phuetz/code-buddy — README mục "Install")
- Chạy được trên mọi máy có Node: VPS, A1 Flex, điện thoại Termux đều ổn về mặt runtime (nhẹ hơn nhiều so với model local, vì LLM gọi qua provider API/Ollama/OpenRouter).
- **License:** Business Source License 1.1 (không phải open source theo chuẩn OSI — có giới hạn dùng cho production/thương mại cần kiểm tra). (https://github.com/phuetz/code-buddy — badge license trên README và LICENSE file trong repo)

## Verdict — Mạnh deploy được không?
**Được có điều kiện** — Kỹ thuật: mỗi máy trong Tailscale mesh chạy `buddy server` là có ngay một "fleet" P2P mà không cần K8s hay server trung tâm; hợp với yêu cầu chi phí ~0 và điện thoại Termux (Node chạy được). Nhưng: (1) đây là **coding agent**, không phải orchestrator multi-agent tổng quát — điều phối task cross-device còn thô (remote tool chỉ read-only, không có task routing/capability matching); (2) **license BSL 1.1 không phải MIT/Apache** — nếu Mạnh chỉ dùng nội bộ/cá nhân thì ổn, nhưng cần đọc kỹ điều khoản trước khi dùng cho việc kiếm tiền; (3) không phải công cụ được thiết kế cho phone farm/ADB — chỉ là "mỗi node một agent" chứ không "một conductor điều phối nhiều máy".

## Nguồn (URL cụ thể đã đọc)
- https://github.com/phuetz/code-buddy (README chính — fleet hub, install, license badge; 48 stars, 6.059 commit, commit 7b77f46ac74f922d8778a660c1270da746bd0574)
