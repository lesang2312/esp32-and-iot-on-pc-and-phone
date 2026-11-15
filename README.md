# 📟 ESP32 OLED Buddy v3

Thiết bị phụ nhỏ cho bàn làm việc: **ESP32 + màn OLED 128x64 + 2 nút bấm**. Nó vừa là đồng hồ, ghi chú, trợ lý lệnh Linux, vừa là **màn hình phụ và bộ điều khiển cho máy tính (qua USB)** và **điện thoại (qua WiFi)**.


---

## 1. Linh kiện

| Linh kiện | Ghi chú |
|---|---|
| ESP32 DevKit (esp32dev, chip CP2102 hoặc CH340) | bo mạch chính |
| Màn OLED SSD1306 0.96" 128x64, giao tiếp I2C, 4 chân | địa chỉ I2C `0x3C` |
| 2 nút bấm nhỏ (tact switch) | |
| Cáp micro-USB / USB-C **có truyền dữ liệu** | cáp chỉ sạc sẽ không dùng được |
| *(tuỳ chọn)* còi chủ động 3.3–5V | cảnh báo bằng tiếng bíp |
| *(tuỳ chọn)* quang trở LDR + điện trở 10 kΩ | tự chỉnh độ sáng màn |
| *(tuỳ chọn)* cảm biến AHT20 hoặc BMP280/BME280 | nhiệt độ, độ ẩm, áp suất |
| *(tuỳ chọn)* 2 điện trở bằng nhau (vd 100 kΩ) | đo pin Li-ion |

## 2. Đấu chân

### Bắt buộc

| Linh kiện | Chân linh kiện | Chân ESP32 |
|---|---|---|
| OLED | VCC (VDD) | **3V3** |
| OLED | GND | **GND** |
| OLED | SDA | **D21** (GPIO21) |
| OLED | SCK / SCL | **D22** (GPIO22) |
| Nút A | một chân | **D4** (GPIO4) |
| Nút A | chân còn lại | **GND** |
| Nút B | một chân | **D18** (GPIO18) |
| Nút B | chân còn lại | **GND** |

Nút dùng điện trở kéo lên bên trong ESP32, nên **không cần thêm điện trở**. Bấm nút = nối chân xuống GND.
Nếu bạn nối nút vào 3V3 thay vì GND, bỏ dấu `;` trước `-DBTN_ACTIVE_HIGH` trong `platformio.ini`.

```
              ┌──────────── ESP32 ────────────┐
   OLED       │                               │
  ┌──────┐    │                               │
  │ VCC ─┼────┤ 3V3                           │
  │ GND ─┼────┤ GND                           │
  │ SDA ─┼────┤ D21                           │
  │ SCK ─┼────┤ D22                           │
  └──────┘    │                               │
              │                               │
  Nút A  ─────┤ D4          (chân kia → GND)  │
  Nút B  ─────┤ D18         (chân kia → GND)  │
              └───────────────────────────────┘
```

### Tuỳ chọn (sửa số chân trong `src/config.h`, để `-1` là không dùng)

| Linh kiện | Cách nối | Cấu hình |
|---|---|---|
| Còi chủ động | `+` → chân GPIO, `−` → GND | `PIN_BUZZ` |
| Quang trở LDR | 3V3 → LDR → chân ADC → điện trở 10 kΩ → GND (chân ADC1: 34, 35, 36 hoặc 39) | `PIN_LDR` |
| Đo pin Li-ion | pin+ → R → chân ADC → R → GND (2 điện trở bằng nhau), chân ADC1 | `PIN_VBAT` |
| AHT20 / BMP280 / BME280 | VCC → 3V3, GND → GND, SDA → D21, SCL → D22 (**dùng chung đường I2C với OLED**) | tự nhận, không cần cấu hình |

## 3. Nạp firmware

```
pio run -t upload
```

- Cần quyền mở cổng USB serial: Arch `sudo usermod -aG uucp $USER`, Debian/Ubuntu `sudo usermod -aG dialout $USER`, rồi đăng nhập lại.
- **Đóng `pio device monitor`** (và nhả cổng của `buddyd.py`) trước khi nạp.
- Bluetooth tắt mặc định (`-DENABLE_BT=0`) vì làm đầy bộ nhớ flash.

## 4. Điều khiển bằng 2 nút

| Thao tác | Việc |
|---|---|
| Bấm A (D4) | **LÊN** |
| Bấm B (D18) | **XUỐNG** |
| Giữ A khoảng 0,6 giây | **CHỌN (OK)** |
| Giữ B khoảng 0,6 giây | **QUAY LẠI** |

Màn hình chuột, soạn chữ và game dùng chế độ phản hồi tức thì khi bấm:

- **Game, soạn chữ:** giữ B để thoát (hoặc giữ cả hai nút 0,7 giây).
- **Chuột:** giữ B dùng để di chuyển, nên thoát bằng **giữ cả hai nút 0,7 giây** (hoặc chọn công cụ "THOAT").

## 5. Màn hình và tính năng

### 5.1 Màn hình chính
4 kiểu, bấm A/B để đổi: hình nền anime + giờ, số lớn, đồng hồ kim, bảng tổng hợp (giờ, thời tiết, CPU/RAM PC, nhạc, việc cần làm, báo thức). Giữ A để vào menu.

### 5.2 Menu

```
MENU
├─ Man hinh phu PC        ← màn phụ máy tính (4 trang)
├─ Dieu khien PC >        ← chuột, bàn phím, trình chiếu, phím tắt, lệnh PC, khóa máy, webhook
├─ Lenh Linux >           ← tra cứu lệnh (190 lệnh), meo ngẫu nhiên, hỏi PC man/tldr
├─ Dien thoai             ← pin điện thoại, clipboard, tìm điện thoại
├─ Ghi chu · Soan van ban · Doc file txt · Viec can lam
├─ Hen gio >              ← bấm giờ, Pomodoro, đếm ngược, báo thức
├─ Tro choi >             ← Dino, Rắn, Flappy, Pong, Phá gạch
├─ Tien ich >             ← xúc xắc, đồng xu, quả cầu bi 8, đo phản xạ
├─ Thoi tiet · Cam bien · Mang & BT · Mat chop
├─ Quan ly file · Canh bao & auto · Cai dat
└─ Thong tin · Test nut bam
```

### 5.3 Màn hình phụ PC (cần `buddyd.py`)
Bấm A/B để đổi trang; OK để phát/dừng nhạc hoặc tắt tiếng.

| Trang | Nội dung |
|---|---|
| 1 | CPU có đồ thị, RAM, swap, ổ đĩa, nhiệt độ, tải, uptime |
| 2 | Tốc độ mạng tải xuống/tải lên, đồ thị 2 chiều |
| 3 | Pin máy, âm lượng, độ sáng |
| 4 | Bài nhạc đang phát, cửa sổ đang mở, workspace |

Màn hình không tự tắt khi đang nối PC.

### 5.4 Điều khiển PC
- **Chuột USB:** mũi tên tự xoay vòng, **giữ B để đi** theo hướng mũi tên (giữ lâu thì nhanh dần). Bấm A để đổi công cụ: Đi · Trái · Phải · Cuộn xuống · Cuộn lên · Kéo · Thoát. Tốc độ chỉnh được ở "Canh bao & auto".
- **Gõ bàn phím:** soạn chữ trên ESP rồi gõ lên cửa sổ đang mở của PC.
- **Trình chiếu / nhạc:** slide trước/sau, F5/Esc, trang, tua video, phát/dừng, bài trước/sau, âm lượng, màn đen. Giữ A để đổi chế độ.
- **Phím tắt PC:** 28 mục (copy, paste, undo, Alt+Tab, khóa máy, chụp màn hình, âm lượng, độ sáng…).
- **Chạy lệnh PC:** chọn lệnh trong danh sách cho phép, kết quả hiện trên OLED.
- **Webhook:** gọi địa chỉ HTTP (nhà thông minh, IFTTT, ntfy…).

### 5.5 Lệnh Linux
- **Tra lệnh:** khoảng 190 lệnh (file, văn bản, hệ thống, mạng, gói phần mềm, git, Hyprland/âm thanh, lập trình/ESP32, fish, phím tắt terminal). Tìm theo tên hoặc mô tả, nhiều từ cách nhau bằng dấu cách. **Chạy ngay trên ESP32, không cần PC.**
- **Hỏi PC (man/tldr):** PC tra trang man hoặc tldr, kết quả hiện trên OLED.

### 5.6 Soạn văn bản bằng 2 nút
Một dải 7 ký tự ở giữa màn hình, ký tự đang chọn được tô sáng.

- **A:** chạy sang ký tự kế tiếp (**giữ A** để chạy nhanh).
- **B:** chọn ký tự đang sáng.
- Cuối dải có `<-` (xóa 1 ký tự) và nút đổi bảng: chữ thường → CHỮ HOA → số → ký hiệu → từ hay dùng (`sudo `, `git `, `ls `, `--help`, `http://`…).
- **Giữ B:** hoàn tác ký tự vừa chọn và mở menu: Xong · Lưu file .txt · Gõ lên PC · Chép vào clipboard PC · Xuống dòng · Xóa hết · Thoát.
- Dùng chung để nhập từ khóa tìm lệnh, mật khẩu WiFi, ghi chú.

### 5.7 Điện thoại
Hiện pin điện thoại, clipboard gửi từ trang web, nút **Tìm điện thoại** (trang web trên điện thoại sẽ kêu và rung). Điện thoại cần đang mở trang web của ESP32.

### 5.8 Cảm biến, cảnh báo, tự động
- **Cảm biến:** nhiệt độ chip hoặc AHT20/BMP280, độ ẩm, áp suất, độ sáng, pin, đồ thị 30 phút.
- **Cảnh báo (popup + còi nếu có):** CPU > 90%, RAM > 90%, pin PC < 20%, pin điện thoại < 15%, ESP quá nóng, mất kết nối PC. Mỗi loại tối đa 5 phút một lần.
- **Tự động:** chỉnh độ sáng theo quang trở, chế độ ban đêm 23h–6h tối nhất.

### 5.9 Mạng & kết nối
Quét WiFi (chọn mạng, nhập mật khẩu ngay trên thiết bị), đồng bộ giờ NTP, tải thời tiết, xem IP, PIN trang web. Chưa có WiFi thì ESP phát mạng riêng `ESP32-Buddy` (mật khẩu `12345678`, địa chỉ `http://192.168.4.1`).

### 5.10 Các tính năng khác
Ghi chú, danh sách việc cần làm (đồng bộ từ shell), đọc file `.txt` (nhớ vị trí đọc), thời tiết Open-Meteo, bấm giờ, Pomodoro, đếm ngược, báo thức, 5 game, mắt chớp, quản lý file, đổi hình nền 128x64.

## 6. Trang web trên điện thoại

Mở `http://buddy.local` (hoặc địa chỉ IP của ESP32) khi điện thoại cùng mạng WiFi.

| Tab | Chức năng |
|---|---|
| 🏠 Tổng quan | vòng CPU/RAM/pin/âm lượng, nhạc, thời tiết, cảm biến |
| 🖱 Điều khiển | touchpad (chạm = chuột trái, 2 ngón chạm = chuột phải, 2 ngón kéo = cuộn), bàn phím tiếng Việt có dấu gõ lên PC, trình chiếu, âm lượng, độ sáng |
| ✍️ Soạn | ghi chú, popup, gửi file `.txt` sang ESP32, nút **Gửi lên PC**, ảnh nền OLED |
| ⌨️ Lệnh | lệnh PC, tra cứu lệnh Linux, hỏi man/tldr, webhook |
| ⚙️ Cài đặt | độ sáng, đảo màu, cảnh báo, báo thức, WiFi, vị trí thời tiết |

**PIN:** trang điều khiển được cả PC nên có PIN (4 số ngẫu nhiên tạo lần đầu, xem trên OLED ở Menu › Mạng & BT).
- Đặt PIN: `python3 buddy.py pin 123456`
- Tạo ngẫu nhiên: `python3 buddy.py pin new`
- Tắt PIN: `python3 buddy.py pin off`

Một số trình duyệt không cho đọc pin điện thoại trên trang `http://`. Khi đó trang vẫn dùng được, chỉ không hiện phần trăm pin.

## 7. Phía máy tính (`buddyd.py`)

Chương trình nền, chỉ dùng thư viện chuẩn Python 3.

```
python3 tools/buddyd.py
```

Nó đẩy CPU/RAM/nhiệt/pin/mạng/nhạc/cửa sổ sang OLED mỗi 2 giây, đồng bộ giờ, Pomodoro và việc cần làm, đồng thời thực hiện các yêu cầu từ ESP32 (chuột, phím, lệnh, tra man). Nó tự thoát khi shell đóng hoặc sau 90 giây không thấy ESP32 (`--idle-exit`).

### Chuột và bàn phím
Ưu tiên `/dev/uinput` cho chuyển động mượt:

```
echo 'KERNEL=="uinput", GROUP="input", MODE="0660"' | sudo tee /etc/udev/rules.d/99-uinput.rules
echo uinput | sudo tee /etc/modules-load.d/uinput.conf
sudo usermod -aG input $USER      # rồi đăng nhập lại
```

Không có `uinput` thì dùng `wlrctl` (chuột Wayland), `wtype` (phím và chữ có dấu), `xdotool` (X11) hoặc `ydotool`. Hành động nhanh dùng `wpctl`/`pactl`, `playerctl`, `brightnessctl`, `grim`.

### Lệnh PC được phép chạy
Mặc định chỉ có các lệnh đọc thông tin (uname, free, df, ip, top, sensors, journal…). Thêm lệnh riêng bằng cách sao chép `tools/buddy-commands.example.json` thành `~/.config/quickshell/study/buddy-commands.json`. Lệnh được khai báo dạng danh sách đối số, **không chạy qua shell**.

### buddy.py (dòng lệnh)
```
buddy.py help
buddy.py note "chữ"        buddy.py wall anh.png     buddy.py put file.txt
buddy.py mouse 10 0        buddy.py key ctrl+c       buddy.py type "xin chào"
buddy.py act vol +5        buddy.py man grep         buddy.py cheat "dung luong"
buddy.py set bright 3      buddy.py pin               buddy.py daemon status
```

## 8. Bảo mật

- Trang web điều khiển được PC nên **nên bật PIN**. Đừng bật PIN rỗng trên mạng WiFi không tin cậy.
- Dữ liệu từ PC (`HELLO`, `PCS2`, `VIEW*`) và lệnh `PIN` chỉ nhận qua cáp USB, không nhận qua WiFi.
- ESP32 chỉ gửi **yêu cầu**; chương trình trên PC quyết định lệnh nào được chạy.
- Chữ gõ lên PC đi vào cửa sổ đang focus, cẩn thận khi đang mở terminal.
- Webhook HTTPS không kiểm tra chứng chỉ, chỉ dùng với dịch vụ trong mạng nhà.

## 9. Giới hạn đã biết

- Màn OLED chỉ hiển thị ASCII, chữ tiếng Việt bị bỏ dấu (riêng chữ gõ từ điện thoại lên PC giữ nguyên dấu).
- Bluetooth tắt mặc định.
- Chưa thử trên phần cứng thật trong môi trường phát triển.

## 10. Cấu trúc thư mục

```
esp32-oled-buddy/
├─ platformio.ini
├─ README.md
├─ src/
│  ├─ main.cpp     vòng lặp chính
│  ├─ config.h     chân, enum màn hình, cấu hình
│  ├─ core.h       cài đặt, giờ, nút bấm, hàm vẽ chung
│  ├─ apps.h       ghi chú, việc cần làm, đọc file, hẹn giờ, thời tiết, mạng, cài đặt
│  ├─ games.h      5 game
│  ├─ net.h        WiFi, NTP, thời tiết, web, PIN
│  ├─ pc.h         liên kết PC/điện thoại, cảm biến, cảnh báo
│  ├─ editor.h     soạn văn bản 2 nút
│  ├─ cheat.h      cơ sở dữ liệu lệnh Linux + tìm kiếm
│  ├─ smart.h      màn phụ PC, chuột, trình chiếu, phím tắt, webhook, cảm biến
│  ├─ page.h       trang web điện thoại
│  ├─ web2.h       API web
│  ├─ commands.h   lệnh USB/web
│  └─ ui.h         menu, màn hình chính, xử lý nút
└─ tools/
   ├─ buddyd.py    chương trình nền trên PC
   ├─ buddy.py     công cụ dòng lệnh
   └─ buddy-commands.example.json
```
