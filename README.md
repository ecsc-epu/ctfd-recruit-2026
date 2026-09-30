# ctfd-recruit-2026

CTF platform for the **EPU Cybersecurity Club (ECSC)** 2026 recruitment round: [CTFd 3.8.7](https://github.com/CTFd/CTFd)
with the `ecsc` theme (1998 keygen / space rainbow / horror look, built to stay light when everyone joins at once).

## Chạy trên server

Server chỉ cần Docker. Theme đã build sẵn, không cần Node.js.

```bash
git clone git@github.com:ecsc-epu/ctfd-recruit-2026.git
cd ctfd-recruit-2026
docker compose up -d --build
```

Mở **http://IP-server** → trang **Setup** → tạo tài khoản admin → **Finish**. Trang quản trị: `/admin`.

Trước khi mở cho người chơi:

- **Đổi mật khẩu database** trong `docker-compose.yml` (`MARIADB_PASSWORD`, `MARIADB_ROOT_PASSWORD` và mật khẩu trong
  `DATABASE_URL`, cả ba phải khớp). Đổi **trước lần chạy đầu tiên**.
- Người chơi **không tự đăng ký được**. Admin tạo tài khoản ở Admin Panel → Users → **+**, hoặc nhập cả danh sách
  ở Admin Panel → Config → Backup → **Import CSV** (loại *Users*, các cột `name,email,password`).
- Dữ liệu nằm trong `.data/` (database, file đề, secret key). Nhớ sao lưu thư mục này. Nó không bao giờ bị đưa lên git.

| Việc | Lệnh |
|---|---|
| Xem trạng thái | `docker compose ps` |
| Xem log | `docker compose logs -f ctfd` |
| Dừng / bật lại | `docker compose stop` / `docker compose start` |
| Cập nhật bản mới | `git pull && docker compose up -d --build` |

Hướng dẫn chi tiết (sửa giao diện, danh sách thay đổi so với CTFd gốc, xử lý lỗi): **[HUONG-DAN.md](HUONG-DAN.md)**.

## Nguồn

Dựa trên [CTFd](https://github.com/CTFd/CTFd) © CTFd LLC, giấy phép Apache-2.0 (xem [LICENSE](LICENSE)).
Các thay đổi so với CTFd 3.8.7 được liệt kê trong `HUONG-DAN.md`.
