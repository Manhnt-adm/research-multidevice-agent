# hyperspaceai/agi
- URL: https://github.com/hyperspaceai/agi
- Stars: 2072 (tại thời điểm đọc)
- Đánh giá ngày: 23/10/2026

## Là gì (2 dòng)
Mạng nghiên cứu AGI **phi tập trung công cộng** (P2P, libp2p/GossipSub): hàng nghìn agent tự trị chạy thí nghiệm ML, gossip kết quả cho nhau, lưu leaderboard bằng CRDT, archive lên GitHub. Đây là **mạng nghiên cứu**, không phải nền tảng orchestrate agent cá nhân.

## Kiến trúc multi-device (chi tiết vừa đủ)
- "Distributed/P2P gossip" ở đây là **thật về mặt kỹ thuật mạng**: libp2p, 6 bootstrap node toàn cầu, GossipSub broadcast kết quả thí nghiệm (~1s) → CRDT leaderboard (~2 phút) → GitHub archive (~5 phút); đã demo 32 node độc lập cùng train 1 model trong 24h (DiLoCo + SparseLoCo + Parcae, nén gradient 195×).
- Điểm liên quan nhất tới Mạnh: **Pods — Private AI Clusters**. `hyperspace pod create "my-lab"` + `hyperspace pod invite` → các máy (cài CLI) tạo thành **mesh riêng**: distributed inference (query route tới member có model tốt nhất đang load, GGUF tới 32B+), **shared providers** (pool chung key OpenRouter/Groq/Together với budget per-member), Pod Capsule (`.tar.gz` mã hóa AES-256, migrate bằng `docker compose up`).
- Agent join bằng browser (agents.hyper.space, model <4B qua WebGPU) hoặc CLI (`curl -fsSL https://agents.hyper.space/api/install | bash` — cài uv, tải binary, auto-start qua systemd/LaunchAgent/Task Scheduler).
- **Nhưng**: agent trên mạng chạy **vòng lặp nghiên cứu của mạng** (hypothesis → training → paper → peer critique), không chạy task do Mạnh giao, không có tool điều khiển thiết bị, không có khái niệm coordinator/worker cho công việc cá nhân. Capability "Orchestration" (+5% points) chỉ là "multi-step task decomposition + routing" trong ngữ cảnh research network, chưa xác minh dùng được cho task cá nhân.
- Kinh tế: hệ điểm (presence + work points) và blockchain "Hyperspace A1" riêng — yếu tố token/crypto cần cân nhắc.

## Yêu cầu triển khai
- CLI qua script cài đặt (bash), binary theo OS (đã thấy `darwin-arm64`; bản Linux tồn tại nhưng **chưa xác minh linux-arm64** — quan trọng vì A1 Flex của Mạnh là ARM).
- Auto-start cần systemd/LaunchAgent/Task Scheduler — **Termux trên S8+/S20FE không có systemd, khả năng chạy chưa xác minh được**.
- Join mặc định là vào mạng công cộng; Pod riêng cần tạo chủ động.

## Docs có đủ để dựng không?
- README chi tiết về join network, pods, training, CLI vs browser; có cộng đồng viết guide thực tế về Pods.
- Thiếu cho use-case của Mạnh: không có tài liệu "dùng Pods làm fleet chạy task cá nhân" (vì đó không phải mục đích của dự án).

## Verdict — Mạnh deploy được không?
**Loại.** P2P/distributed là thật, nhưng đây là mạng nghiên cứu AGI công cộng — agent chạy thí nghiệm ML của mạng để kiếm points, không phải agent làm việc cho Mạnh; không có orchestration task cá nhân, không điều khiển được thiết bị. Điểm **đáng chú ý nhất**: ý tưởng **Pods** (mesh máy riêng + pool chung key OpenRouter + inference phân tán) rất đáng tham khảo cho kiến trúc của Mạnh, nhưng triển khai Hyperspace cho mục đích đó thì sai tool. Cân nhắc thêm: cài bằng `curl | bash` binary của bên thứ ba, yếu tố points/blockchain, và chưa xác minh hỗ trợ linux-arm64 (A1 Flex) lẫn Termux (2 điện thoại).

## Nguồn (URL cụ thể đã đọc)
- https://github.com/hyperspaceai/agi (README: Pods — Private AI Clusters, distributed training 32 node, 9 network capabilities, join via browser/CLI, blockchain Hyperspace A1)
- https://deepwiki.com/hyperspaceai/agi/1.2-getting-started (cài CLI: detect OS/GPU, tải binary darwin-arm64, systemd/LaunchAgent/Task Scheduler)
- https://deepwiki.com/hyperspaceai/agi/2.2-network-capabilities (distributed inference trong Pod, shared providers, local API /chat/completions)
- https://ianloe.medium.com/pool-your-old-machines-into-an-ai-cluster-a-practical-guide-to-hyperspace-pods-2e1552e14e07 (guide thực tế dựng Pod: install 1 dòng, pod lên trong ~10 phút)
