# Open-RMF (open-rmf/rmf_demos)

- URL: https://github.com/open-rmf/rmf_demos
- Stars: 613 (tại thời điểm đọc)
- Đánh giá ngày: 23/10/2026

## Là gì (2 dòng)
Middleware mở (Open Robotics Middleware Framework) giúp các fleet robot KHÁC HÃNG tương tác được với nhau trong cùng cơ sở, quản lý task allocation và giải xung đột tài nguyên chung (làn đường, thang máy, cửa...). Repo này là bộ demo (hotel, office, airport, clinic...), chạy trên ROS 2 + Gazebo, Ubuntu 24.04.

## Pattern điều phối đáng học cho fleet agent của Mạnh
Có 2 pattern ở tầng ý tưởng: (1) "fleet adapter" — mỗi loại thực thể (kể cả hệ không điều khiển được, chỉ đọc vị trí như xe caddy lái tay → classify `read_only`) có một adapter chuẩn nói chuyện với scheduler trung tâm, robot không cần cùng hãng/API; (2) scheduler trung tâm de-conflict tài nguyên dùng chung thay vì để từng fleet tự tranh giành. Nhưng triển khai là cho robot vật lý: bản đồ làn đường, traffic editor, lift/door — không có ánh xạ 1-1 sang tranh tài nguyên phần mềm của Mạnh (API rate limit, file, port). Phần chi tiết "Task Dispatching in Open-RMF" trong README chưa đọc được (bị giới hạn request), nên chưa xác minh được cơ chế dispatch cụ thể.

## Verdict
**Không hợp để áp dụng trực tiếp** — nặng đô (ROS 2 + Gazebo, Ubuntu, infra vật lý), pattern giải xung đột gắn chặt với không gian vật lý, không mang sang fleet agent phần mềm chi phí ~0 của Mạnh; chỉ tham khảo ý tưởng "adapter chuẩn + scheduler trung tâm de-conflict".

## Nguồn (URL cụ thể đã đọc)
- https://github.com/open-rmf/rmf_demos (README: giới thiệu Open-RMF, task allocation & conflict resolution, các demo world, fleet adapter)
