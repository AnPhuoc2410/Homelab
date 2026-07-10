# What's Up Docker (WUD)

**What's Up Docker (WUD)** là một công cụ giám sát cập nhật image Docker cho homelab. Mục tiêu chính của WUD là giúp bạn biết khi nào một container hoặc một image có phiên bản mới, sau đó tự động gửi cảnh báo hoặc kích hoạt các hành động liên quan đến quy trình cập nhật.

Trong homelab này, WUD đóng vai trò như một lớp theo dõi tập trung cho các stack Docker Compose. Thay vì phải kiểm tra thủ công từng service, WUD sẽ quét image theo lịch và có thể thông báo qua Discord hoặc kích hoạt các trigger riêng cho từng stack.

---

## Mục đích sử dụng trong homelab

WUD được thêm vào homelab để giải quyết các nhu cầu sau:

1. **Theo dõi version mới của image Docker**: phát hiện khi image đang chạy đã có bản cập nhật mới trên registry.
2. **Giảm việc kiểm tra thủ công**: không cần đăng nhập từng service để xem có cập nhật hay không.
3. **Tự động hóa thông báo**: đẩy thông báo lên Discord để bạn biết ngay khi có update đáng chú ý.
4. **Hỗ trợ quy trình cập nhật theo stack**: với Docker Compose trigger, WUD có thể gắn với một stack cụ thể để phục vụ quy trình backup, prune hoặc các bước cập nhật khác.
5. **Giữ homelab có tính chủ động**: cập nhật được quản lý theo lịch, có giám sát, thay vì phản ứng sau khi service lỗi hoặc bị lỗi thời quá lâu.

Nói ngắn gọn, WUD là công cụ “canh cửa” cho các container của bạn: nó không thay bạn triển khai update, nhưng giúp bạn biết chính xác lúc nào cần làm việc đó và có thể nối vào các bước tự động tiếp theo.

---

## WUD làm gì?

WUD thường thực hiện các bước sau:

1. **Quét Docker host** qua Docker socket để xem các container/image đang chạy.
2. **So sánh tag hoặc digest** của image hiện tại với bản mới trên registry.
3. **Đánh dấu container có bản cập nhật** nếu phát hiện image mới hơn.
4. **Kích hoạt trigger** theo cấu hình để gửi thông báo hoặc thực thi hành động.
5. **Lặp lại theo lịch** để việc theo dõi diễn ra tự động.

Với cấu hình hiện tại, WUD đang được thiết kế để:

* theo dõi theo lịch định kỳ,
* gửi cảnh báo qua Discord,
* và liên kết với một stack Docker Compose cụ thể như `n8n`.

---

## Cấu hình hiện tại

File `docker-compose.yml` trong thư mục này cho thấy WUD đang chạy với các thành phần chính sau:

| Thành phần | Vai trò |
|---|---|
| `getwud/wud` | Image chính của WUD |
| `/var/run/docker.sock` | Cho phép WUD đọc thông tin Docker host |
| Cổng `3000` | Web UI của WUD |
| `WUD_WATCHER_LOCAL_CRON` | Lịch quét image tự động |
| `WUD_TRIGGER_DISCORD_*` | Gửi cảnh báo đến Discord webhook |
| `WUD_TRIGGER_DOCKERCOMPOSE_N8N_*` | Gắn trigger cho stack `n8n` |

### Ý nghĩa các biến môi trường chính

* `WUD_LOG_LEVEL=info`: bật mức log thông tin để dễ theo dõi hoạt động.
* `WUD_WATCHER_LOCAL_CRON=0 0 * * *`: quét theo lịch cron, ở đây là mỗi ngày lúc 00:00.
* `WUD_TRIGGER_DISCORD_MYDISCORD_URL=...`: endpoint Discord webhook để nhận thông báo.
* `WUD_TRIGGER_DOCKERCOMPOSE_N8N_FILE=...`: đường dẫn đến file `docker-compose.yml` của stack cần liên kết.
* `WUD_TRIGGER_DOCKERCOMPOSE_N8N_PRUNE=true`: cho phép dọn các resource không còn cần thiết theo luồng cập nhật.
* `WUD_TRIGGER_DOCKERCOMPOSE_N8N_BACKUP=true`: bật cơ chế backup trước khi thao tác với stack.

---

## Lợi ích thực tế

WUD đặc biệt hữu ích nếu bạn:

1. chạy nhiều container từ nhiều thư mục Compose khác nhau,
2. muốn biết ngay khi image mới xuất hiện,
3. ưu tiên vận hành homelab có kiểm soát thay vì kiểm tra rời rạc,
4. đang dùng Discord như kênh thông báo trung tâm,
5. muốn chuẩn bị cho một quy trình update có backup và cleanup đi kèm.

Đối với homelab, đây là một công cụ quan sát rất đáng có vì image Docker thay đổi liên tục, đặc biệt với các dịch vụ như reverse proxy, automation, media stack, hoặc các ứng dụng thường xuyên phát hành bản vá bảo mật.

---

## Cách truy cập

Sau khi container khởi động, giao diện web của WUD thường truy cập tại:

```text
http://<your-server-ip>:3000
```

Tại đây bạn có thể xem trạng thái các image, các update khả dụng, và các trigger đã được cấu hình.

---

## Yêu cầu trước khi chạy

Để WUD hoạt động đúng, cần đảm bảo:

1. Docker host có thể đọc được `/var/run/docker.sock`.
2. Các đường dẫn gắn volume trong Compose là hợp lệ và trỏ đến đúng thư mục stack.
3. Webhook Discord đã được thay bằng URL thật của bạn.
4. Các biến liên quan đến Docker Compose trigger trỏ đúng đến file `docker-compose.yml` cần quản lý.
5. Quyền truy cập thư mục và file đủ để WUD đọc cấu hình khi cần.

---

## Triển khai

Khởi động stack bằng lệnh:

```bash
docker compose up -d
```

Kiểm tra trạng thái container:

```bash
docker compose ps
```

Xem log nếu cần chẩn đoán:

```bash
docker compose logs -f
```

---

## Lưu ý bảo mật

WUD cần quyền đọc Docker socket, nên hãy xem nó như một dịch vụ có mức quyền khá cao trong hệ thống. Chỉ dùng trên máy bạn tin cậy và không công khai Web UI ra Internet nếu không có lớp bảo vệ phù hợp.

Ngoài ra, không nên để webhook Discord hoặc các biến nhạy cảm nằm nguyên dạng placeholder nếu repo được chia sẻ công khai.

---

## Kết luận

WUD trong homelab này là công cụ giám sát cập nhật image Docker và tự động hóa thông báo/các bước liên quan đến update. Nó giúp bạn giữ các service luôn trong tầm kiểm soát, giảm việc kiểm tra thủ công, và tạo nền tảng cho quy trình bảo trì container có kỷ luật hơn.
