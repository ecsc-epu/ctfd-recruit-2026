# CTFd ECSC: hướng dẫn chi tiết

Repo này là **CTFd 3.8.7** (github.com/CTFd/CTFd, giấy phép Apache-2.0) đã chỉnh cho EPU Cybersecurity Club, kèm theme **`ecsc`**. Theme mang phong cách
của web tuyển thành viên (nền đen, chữ chrome, viền máu, cửa sổ Windows 98, chấm dither 1998, màu hồng),
nhưng **không có 3D, nhạc, kéo thả hay animation**. Mục tiêu chính là **nhẹ và nhanh** khi cả trăm người
cùng vào trong lúc thi.

```
ctfd-recruit-2026/
├── README.md               ← cách đưa lên server (ngắn)
├── HUONG-DAN.md            ← file này
├── docker-compose.yml      ← CTFd + MariaDB + Redis + nginx
├── CTFd/themes/ecsc/       ← theme ECSC (bản sao của theme "core" rồi sửa lại, static/ đã build sẵn)
├── CTFd/themes/admin/      ← giao diện Admin Panel (giữ nguyên)
└── CTFd/constants/themes.py   ← đã đổi theme mặc định thành "ecsc"
```

---

## 0. Giao diện nằm ở đâu

CTFd tự phục vụ cả trang người chơi lẫn trang quản trị, **cùng một địa chỉ**, không có phần giao diện chạy riêng:

| Giao diện | Chạy thật (mục 1, `docker compose`) | Chạy thử 1 container (mục 2) |
|---|---|---|
| Trang người chơi (theme `ecsc`: đề, nộp flag, bảng điểm) | http://IP-server (cổng 80) | http://localhost:8000 |
| Trang quản trị (tạo đề, người chơi, cài đặt) | http://IP-server/admin | http://localhost:8000/admin |

Lần đầu mở sẽ ra màn **Setup** để tạo tài khoản admin. Vào `/admin` bằng tài khoản đó.
Trang quản trị giữ giao diện gốc của CTFd, theme ECSC chỉ áp cho phía người chơi.

## 1. Chạy trên server (MariaDB + Redis + nginx)

Server chỉ cần Docker (có sẵn `docker compose`). Không cần Node.js hay Python: theme đã build sẵn trong repo.

```bash
git clone git@github.com:ecsc-epu/ctfd-recruit-2026.git
cd ctfd-recruit-2026
docker compose up -d --build
```

Lần đầu mất vài phút để build image (cần mạng để tải gói Python). Sau đó mở **http://IP-server**, trang
**Setup** hiện ra:

1. Điền tên CTF, mô tả, chế độ (Users hoặc Teams), tài khoản admin.
2. Theme `ecsc` đã được chọn sẵn.
3. Bấm **Finish**.

- Web chạy ở cổng **80**, qua nginx đã bật **gzip** (CSS và JS nhỏ đi khoảng 5 lần). Cổng 8000 của CTFd
  **không mở ra ngoài**: CTFd tin header `X-Forwarded-For` của nginx, mở thẳng cổng 8000 thì người chơi giả được IP.
- Dữ liệu nằm trong thư mục `.data/` cạnh `docker-compose.yml` (database, file đề, log, secret key).
  **Nhớ sao lưu** thư mục này. Nó không bao giờ bị đưa lên git.
- Secret key (dùng để ký phiên đăng nhập) tự tạo **một lần** ở `.data/CTFd/secret/key`, nên khởi động lại hay
  build lại image cũng không làm người chơi bị đăng xuất.
- **Trước khi mở cho người chơi:**
  - Đổi mật khẩu database trong `docker-compose.yml`: `MARIADB_PASSWORD`, `MARIADB_ROOT_PASSWORD` và mật khẩu trong
    `DATABASE_URL` phải khớp nhau. Đổi **trước lần chạy đầu tiên**, vì MariaDB chỉ đọc các biến này khi tạo database.
  - Nên chạy sau HTTPS (ví dụ Cloudflare hoặc thêm chứng chỉ vào `conf/nginx/http.conf`).
- Tăng `WORKERS` trong `docker-compose.yml` nếu server nhiều nhân (mỗi worker khoảng 1 nhân CPU).

Dừng: `docker compose down` (dữ liệu vẫn còn trong `.data/`).

Cập nhật khi repo có bản mới: `git pull` rồi `docker compose up -d --build`. Nếu thay đổi chỉ nằm trong theme
thì xem mục 8, phần "vừa sửa theme".

## 2. Chạy thử trên máy (1 container, SQLite)

Cần Docker.

```bash
cd ctfd-recruit-2026
docker build -t ecsc-ctfd .
docker run -d --name ecsc-ctfd -p 8000:8000 ecsc-ctfd
```

Mở http://localhost:8000. Trang **Setup** hiện ra:

1. Điền tên CTF, mô tả, chế độ (Users hoặc Teams), tài khoản admin.
2. Ở tab **Theme**, chọn `ecsc` (đã chọn sẵn vì là theme mặc định).
3. Bấm **Finish**.

> **Nhớ dấu chấm `.` ở cuối lệnh `docker build`.** Dấu chấm nghĩa là "build từ thư mục hiện tại".
> Thiếu nó sẽ báo lỗi `'docker build' requires 1 argument`.

Cách này lưu dữ liệu **bên trong container**: `docker stop` / `docker start` thì dữ liệu vẫn còn,
nhưng `docker rm` là mất hết. Chỉ dùng để thử. Các lệnh quản lý xem ở mục 7.

## 3. Chọn hoặc đổi theme sau khi đã cài

Chỉ còn một theme cho người chơi là `ecsc` (theme gốc `core` và `core-deprecated` đã bị xóa), nên không cần chọn gì.
Nếu Admin Panel → **Config** → **Themes** đang để tên khác thì chọn lại `ecsc` → **Update**.
Trang quản trị (Admin Panel) giữ nguyên giao diện gốc của CTFd, theme chỉ đổi phần người chơi thấy.

Logo, favicon và banner: Admin Panel → Config → **Logo / Theme**. Theme đã có sẵn favicon con ong ECSC.

## 3b. Tài khoản người chơi: chỉ admin tạo

Người chơi **không tự đăng ký được**. Việc này bị khóa cứng trong code, không phụ thuộc cài đặt:

- Không còn nút Register. Mở `/register` thì báo 404.
- Đăng nhập qua OAuth/MajorLeagueCyber không tự tạo tài khoản mới.
- API tạo người dùng (`POST /api/v1/users`) chỉ admin gọi được, như CTFd gốc.
- Admin Panel → Config → Visibility → Registration chỉ còn lựa chọn `Private`.

Cách tạo tài khoản cho người chơi:

- **Từng người:** Admin Panel → **Users** → nút **+** → điền tên, email, mật khẩu → Submit.
- **Cả danh sách (ví dụ các bạn đã qua vòng đơn):** Admin Panel → **Config** → **Backup** → **Import CSV**,
  chọn loại *Users*. File CSV có các cột `name,email,password` (mỗi người một dòng). Sau đó gửi tài khoản cho
  từng người. Người chơi tự đổi mật khẩu ở trang Settings.

Các file đã sửa: `CTFd/utils/decorators/visibility.py`, `CTFd/utils/config/visibility.py`,
`CTFd/utils/config/integrations.py`, `CTFd/views.py`, `CTFd/forms/setup.py`, `CTFd/forms/config.py`,
`CTFd/constants/setup.py`. Template `register.html` của theme đã bị xóa.
Muốn mở lại đăng ký công khai: lấy lại các file trên từ CTFd 3.8.7 gốc (github.com/CTFd/CTFd, tag `3.8.7`), rồi chọn
Public ở mục Visibility.

---

## 4. Theme ECSC đã thay đổi gì so với theme gốc

| | Theme `core` gốc | Theme `ecsc` |
|---|---|---|
| Biểu đồ điểm (echarts, ~570 KB JS) ở bảng điểm, trang người chơi/đội | có | **bỏ** (chỉ giữ bảng xếp hạng và thanh tỉ lệ đơn giản) |
| Nút chuyển sáng/tối | có | **bỏ**, luôn nền tối |
| Âm thanh thông báo | tải file mp3/webm | **không tải gì** (âm thanh rỗng ngay trong trang) |
| Font | Lato + Raleway | Anton, VT323, Space Mono, tự host, cắt theo bảng chữ (tiếng Việt tải khi cần) |
| Icon FontAwesome | woff2 + ttf | chỉ woff2 |
| Ảnh đi kèm theme | 1,5 MB (có file `ctfd.ai` 1 MB) | 44 KB (logo + favicon ECSC) |
| Hiệu ứng chuyển động, transition | có | **tắt hết** |
| Tổng dung lượng file tĩnh của theme | 4,3 MB | **1,1 MB** |
| Nhãn hiệu CTFd ("Powered by CTFd", link ctfd.io/mạng xã hội CTFd trên trang chủ mặc định) | có | **bỏ**, thay bằng ECSC. Không có footer (muốn thêm thì nhập HTML ở Admin → Config → Theme → Theme Footer) |
| Ô "đăng ký bản tin CTFd LLC" lúc cài đặt (tick sẵn, gửi email admin ra ngoài) | có | **bỏ** |
| Trang trí | không | ông mặt trời 3D xoay (ảnh WebP động dựng sẵn, 140 KB, không WebGL), ông hải tặc lảm nhảm, nền sao + dải cầu vồng tĩnh |
| Trang Settings (hồ sơ, access token) | form Bootstrap | **cửa sổ keygen 1998**: thanh tiêu đề, banner ECSC, ô mật khẩu nền đen chữ xanh, dòng trạng thái báo lưu thành công/lỗi, nút PATCH! |

Đo thử (Firefox, máy local): mỗi trang người chơi tải khoảng **580 KB, 7–10 request** (chưa gzip). Không trang
nào còn tải echarts. Qua nginx có gzip, phần CSS và JS chỉ còn khoảng 120 KB.

Sửa thêm một lỗi có sẵn trong CTFd gốc: trang hồ sơ người chơi/đội báo lỗi `this.solves is null` khi chưa tải
xong dữ liệu.

**Ông mặt trời và ông hải tặc:** mặt trời là ông mặt trời 3D của web tuyển thành viên, dựng sẵn thành ảnh WebP động
(`assets/img/sun.webp`) nên máy yếu cũng không tốn GPU. Nền đen của ảnh được hòa vào nền trang bằng `mix-blend-mode: screen`. Ông hải tặc ở góc dưới trái nói
câu hợp với từng trang (đề, bảng điểm, hồ sơ, đăng nhập), đổi câu mỗi 25 giây hoặc khi bấm vào ông. Bong bóng
thoại tự ẩn sau 8 giây và bấm xuyên qua được. Phần ông hải tặc chỉ khoảng 9 KB, không thư viện.
Sửa câu nói trong `templates/components/ecsc.html` (danh sách `L` ở cuối file).

Mọi chức năng thi đấu vẫn giữ nguyên: xem đề, nộp flag, gợi ý, tải file đề, bảng điểm tự cập nhật, đội,
thông báo, đổi ngôn ngữ, cài đặt tài khoản.

## 5. Tự sửa giao diện

Toàn bộ phần phong cách ECSC nằm trong **một file**:

```
CTFd/themes/ecsc/assets/scss/includes/_ecsc.scss
```

Màu chính khai báo ở đầu file (`$pink`, `$blood`, `$win`…). Sửa xong thì build lại theme (cần Node.js 18+):

```bash
cd CTFd/themes/ecsc
npm install            # lần đầu. Nếu npm chặn script của esbuild: npm install-scripts approve esbuild && npm rebuild esbuild
npx vite build         # tạo lại thư mục static/
```

Nhớ commit cả thư mục `static/` vừa build: server dùng thẳng thư mục này, không tự build theme.
Trên server: `git pull`, rồi làm theo mục 8, phần "vừa sửa theme".

Các file khác đã chỉnh, nếu cần sửa sâu hơn:

| File | Nội dung |
|---|---|
| `assets/scss/includes/utils/_fonts.scss` | khai báo font |
| `assets/fonts/` | file font woff2 |
| `assets/img/logo.png`, `favicon.ico` | logo và favicon mặc định |
| `assets/js/scoreboard.js`, `users/*.js`, `teams/*.js` | đã bỏ phần vẽ biểu đồ |
| `assets/js/color_mode_switcher.js` | luôn nền tối |
| `templates/base.html` | âm thanh thông báo rỗng |
| `templates/components/navbar.html` | đã bỏ nút chuyển sáng/tối |
| `templates/components/ecsc.html` | ông mặt trời, ông hải tặc và các câu nói |
| `templates/settings.html` | trang Settings dạng cửa sổ keygen 1998 (style ở cuối `_ecsc.scss`, phần `.kg-*`) |
| `assets/img/sun.webp` | ông mặt trời 3D (xem cách dựng lại ở dưới) |
| `CTFd/constants/themes.py` (ngoài theme) | theme mặc định là `ecsc` |
| `templates/setup.html`, `assets/js/setup.js` | bỏ ô đăng ký bản tin CTFd |
| `CTFd/views.py` (ngoài theme) | trang chủ mặc định tạo lúc cài đặt dùng nội dung ECSC |
| `vite.config.js` | chỉ chép font và ảnh cần dùng |
| `docker-compose.yml` (ngoài theme) | không mở cổng 8000 ra ngoài, tự tạo và giữ secret key trong `.data/CTFd/secret` |
| `conf/nginx/http.conf` (ngoài theme) | bật gzip cho CSS, JS, JSON, SVG (CTFd gốc chỉ nén HTML): CSS từ 356 KB còn 75 KB |
| `.gitignore` (ngoài theme) | đưa theme `ecsc` vào git (CTFd gốc bỏ qua mọi theme trừ `core`/`admin`) |

**Dựng lại ông mặt trời 3D** (ví dụ khi đổi màu mặt trời trên web): script nằm trong dự án web tuyển thành viên
(thư mục `media/src`, **không có trong repo này**). Cần Node 18+, Firefox và ffmpeg.

```bash
cd media/src             # trong dự án web tuyển thành viên
npm i puppeteer-core     # lần đầu
OUT=/đường/dẫn/tới/ctfd-recruit-2026/CTFd/themes/ecsc/assets/img/sun.webp node render-sun.mjs
```

Script chụp 64 khung hình ông mặt trời trên nền đen (trang `campaign.html?p=sun-sprite`), rồi ghép thành WebP động
220 px lặp liền mạch. Xong thì `npx vite build` trong thư mục theme như trên.

## 6. Đưa theme sang một bản CTFd khác

Theme là một thư mục độc lập. Chép `CTFd/themes/ecsc/` (bỏ `node_modules/` cho nhẹ) vào thư mục
`CTFd/themes/` của bản CTFd kia, khởi động lại, rồi chọn theme như mục 3. Theme viết cho CTFd **3.8.x**.
Bản CTFd mới hơn có thể đổi template, khi đó lấy theme `core` trong bản mới làm gốc và làm lại các sửa đổi ở mục 5.

## 7. Quản lý Docker

Theo cách chạy trên server ở mục 1 (chạy các lệnh trong thư mục repo):

| Việc | Lệnh |
|---|---|
| Xem trạng thái | `docker compose ps` |
| Dừng tất cả (giữ dữ liệu trong `.data/`) | `docker compose stop` |
| Bật lại | `docker compose start` |
| Tắt và gỡ container (dữ liệu vẫn trong `.data/`) | `docker compose down` |
| Cập nhật sau khi sửa theme | `docker compose up -d --build` |
| Xem log | `docker compose logs -f ctfd` |
| Sao lưu dữ liệu | `docker compose stop` rồi `sudo tar czf ctfd-backup.tgz .data` rồi `docker compose start` |

Theo cách chạy thử ở mục 2 (container tên `ecsc-ctfd`):

| Việc | Lệnh |
|---|---|
| Xem container đang chạy | `docker ps` |
| Xem cả container đã dừng | `docker ps -a` |
| Dừng (giữ dữ liệu) | `docker stop ecsc-ctfd` |
| Bật lại | `docker start ecsc-ctfd` |
| Xem log (Ctrl+C để thoát) | `docker logs -f ecsc-ctfd` |
| Xoá container (**mất dữ liệu**) | `docker rm -f ecsc-ctfd` |
| Cập nhật sau khi sửa theme | `docker build -t ecsc-ctfd .` rồi `docker rm -f ecsc-ctfd` rồi chạy lại lệnh `docker run` ở mục 2 |
| Xoá image cho đỡ tốn ổ cứng | `docker rmi ecsc-ctfd` |

Lỗi "cổng đã bị dùng" (`port is already allocated`): còn một chương trình khác đang giữ cổng 80 (hoặc 8000 khi chạy thử).
Xem bằng `docker ps` rồi dừng nó, hoặc đổi cổng, ví dụ `-p 8080:8000` rồi mở http://localhost:8080.

Cảnh báo `DEPRECATED: The legacy builder is deprecated…` chỉ là cảnh báo, build vẫn chạy.
Muốn hết thì cài buildx: `sudo pacman -S docker-buildx` (Arch) hoặc `sudo apt install docker-buildx-plugin` (Ubuntu).

## 8. Gặp lỗi

- **`Bind for 0.0.0.0:80 failed: port is already allocated` khi chạy `docker compose`:** server đã có web khác
  (Apache, nginx…) giữ cổng 80. Tắt nó đi, hoặc đổi dòng `80:80` của nginx trong `docker-compose.yml` thành
  `8080:80` rồi mở http://IP-server:8080.
- **nginx khởi động lại liên tục, log báo `host not found in upstream "ctfd:8000"`:** thường xảy ra sau một lần
  khởi động lỗi như trên. Tạo lại hai container: `docker compose up -d --force-recreate ctfd nginx`.

- **Đã cài rồi nhưng trang chủ vẫn là nội dung cũ:** trang chủ được tạo **một lần** lúc cài đặt và lưu trong
  database. Sửa ở Admin Panel → **Pages** → trang `index`.
- **Đang chạy bằng `docker compose` và vừa sửa theme:** file `docker-compose.yml` gắn thẳng thư mục mã nguồn vào
  container, nên sau `npx vite build` chỉ cần `docker compose exec cache redis-cli FLUSHALL` rồi
  `docker compose restart ctfd` (không cần build lại image).

- **Popup (đề bài, gợi ý, token) bị tối và không bấm được:** có phần tử bao ngoài (ví dụ `main`) bị đặt `z-index`,
  làm popup nằm dưới lớp nền tối của Bootstrap. Đừng đặt `z-index` cho `main` hoặc `.container`. Ông mặt trời
  được đặt trước `main` trong `base.html`, còn `main` có `position: relative` (không có `z-index`), nên nội dung tự
  nằm đè lên mặt trời. Đừng đưa mặt trời xuống `z-index: -1`, vì ở tầng đó `mix-blend-mode` không xóa được nền
  đen và sẽ hiện ô vuông tối.
- **Lỗi có sẵn của CTFd 3.8.7 ở trang admin (không ảnh hưởng chức năng):** thiếu font Lato nghiêng, thiếu
  `codemirror.min.css` ở Admin → Config, lỗi JS `appendChild` ở Admin → Users → tạo mới (tạo người dùng vẫn chạy).
- **Giao diện không đổi sau khi sửa:** chưa chạy `npx vite build`, hoặc chưa build lại image Docker, hoặc trình
  duyệt còn cache (Ctrl+Shift+R).
- **Console báo lỗi kết nối `/events`:** đó là luồng thông báo trực tiếp bị cắt khi chuyển trang, bình thường.
  Nếu thông báo không hiện ngay, xem lại cấu hình nginx (luồng `/events` cần tắt buffering, file
  `conf/nginx/http.conf` của CTFd đã có sẵn).
- **Muốn quay về giao diện gốc:** theme `core` đã bị xóa khỏi repo này. Chép thư mục `CTFd/themes/core` từ CTFd 3.8.7
  gốc (github.com/CTFd/CTFd, tag `3.8.7`) vào `CTFd/themes/`, khởi động lại, rồi chọn `core` trong
  Admin Panel → Config → Themes.
- **Trang báo lỗi 500 hoặc người chơi bị đăng xuất liên tục:** xem `docker compose logs ctfd`. Nếu thấy lỗi đọc
  `/var/secret/key`, chạy lại `docker compose up -d` để service `permissions` tạo lại key.
- **Trang không có màu, chỉ còn chữ và gạch đầu dòng:** vừa `npx vite build` nhưng CTFd vẫn nhớ tên file CSS cũ.
  Chạy `docker compose exec cache redis-cli FLUSHALL` rồi `docker compose restart ctfd`
  (chạy một container thì build lại image).
