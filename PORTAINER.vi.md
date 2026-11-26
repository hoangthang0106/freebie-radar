# Freebie Radar — cài qua Portainer Git (LAN, một container)

Phiên bản hướng dẫn cho fork: https://github.com/hoangthang0106/freebie-radar

## Cổng theo thứ tự cài đặt

- **SpiderFoot**: host port **18894** (đã cấu hình trước; trạng thái chạy thực tế phải xác minh).
- **Freebie Radar**: host port **18895** → container port **18895**.
- **Không tự vượt lên 18899 chỉ vì các ứng dụng khác mới được dự kiến mà chưa xác nhận đã triển khai.**
- **Cảnh báo**: Fork OpenOSINT cũ cũng được chuẩn bị với host port 18895. Không triển khai đồng thời hai dịch vụ với cùng host IP:port. Khi muốn cài OpenOSINT, kiểm tra port và cấp lại theo hệ thống đang chạy.

## Trước khi cài

Trên đúng máy/VM/LXC Docker (chỉ đọc):

```sh
ss -ltnp '( sport = :18895 )'
docker ps -a --format 'table {{.Names}}\t{{.Ports}}'
docker ps -aq | xargs -r docker inspect --format '{{.Name}} {{json .HostConfig.PortBindings}}'
```

Nếu 18895 đang bị dùng, không Deploy cho tới khi xác định port thực sự trống.

## Portainer → Stacks → Add stack → Git repository

| Trường | Giá trị |
|---|---|
| Name | `freebie-radar` |
| Repository URL | `https://github.com/hoangthang0106/freebie-radar.git` |
| Repository reference | `refs/heads/main` |
| Compose path | `docker-compose.yml` |
| Authentication | None (public repo) |

### Environment variables bắt buộc

| Name | Value |
|---|---|
| `LAN_IP` | IP LAN thực tế thuộc Docker VM/LXC |
| `ADMIN_TOKEN` | Mật khẩu/token quản trị mạnh do bạn tự đặt (nên ≥24 ký tự) |

Không nhập secret vào GitHub, không đưa vào tên Stack. Không cần thêm biến `PORT` ở Portainer, vì cổng nội bộ đã đặt **18895** ngay trong Compose.

Tùy chọn `RADAR_IMAGE` để pin phiên bản upstream; mặc định `zichuanlan/freebie-radar:latest`. Theo quy ước **host port = container port**, không tự đổi `RADAR_HOST_PORT` mà không đồng thời cập nhật cổng lắng nghe, mapping và healthcheck trong Compose.

## Triển khai & xác minh

- Nhấn **Deploy the stack**. Chỉ có service/container `radar`, không Caddy hoặc FlareSolverr.
- Vào Portainer → Containers → Freebie Radar. Kiểm tra Running/Healthy, Published Port `LAN_IP:18895:18895/tcp`.
- Mở `http://LAN_IP:18895` ở điện thoại/máy cùng LAN.
- Quản trị dùng `ADMIN_TOKEN` trong UI; dòng tin/ưu đãi công khai vẫn có thể xem mà không cần đăng nhập.
- Cần xem lỗi: Containers → Logs; hoặc truy cập `http://LAN_IP:18895/health` để kiểm tra phản hồi.

## Bảo mật

- Không NAT port 18895 ra Internet. Nếu cần từ xa, dùng VPN/HTTPS.
- Chạy trên IP LAN cụ thể, `NODE_ENV=production`, `SITE_MODE=selfhost`, `ADMIN_TOKEN` bắt buộc.
- Volume `radar_data` lưu `/data/radar.db`; backup volume trước khi nâng cấp, không xóa volume khi chỉnh Stack.
- Compose sử dụng image tác giả gốc; fork của bạn hiện điều khiển **cấu hình Docker Compose**, không tự động build image từ code GitHub của bạn. Thay source code trong fork không ảnh hưởng image `zichuanlan/freebie-radar:latest` cho tới khi bạn chủ động build image riêng.
- Đừng bật auto-update, thêm dịch vụ proxy, FlareSolverr, Docker socket hoặc mở Internet nếu chưa có yêu cầu.
- Portainer sẽ dùng đúng Compose file trong fork, không cần sao chép YAML sang Web editor.

**Trạng thái: đã sửa mã nguồn Compose trên GitHub; chưa deploy vào Docker host, chưa xác minh port 18895 trống.**
