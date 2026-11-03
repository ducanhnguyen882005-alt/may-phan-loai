# Xem máy phân loại từ xa qua GitHub Pages

`index.html` là trang tĩnh trên GitHub Pages, mở được từ bất cứ đâu:
https://ducanhnguyen882005-alt.github.io/may-phan-loai/

## Trang lấy dữ liệu từ đâu

```
ESP32 (WiFi trường) ──HTTPS, 10 giây/lần──► Firebase Realtime Database ◄──HTTPS── trang GitHub Pages
                                             (gói Spark, miễn phí)
```

Trang **không gọi thẳng ESP32**: trình duyệt chặn trang HTTPS gọi vào
`http://192.168.x.x`, còn WiFi trường thì chặn mọi kết nối gọi vào board.
Cả ESP32 lẫn trang web cùng gọi ra Firebase.

- Số liệu trễ tối đa khoảng **10 giây**. Trang tự tải lại mỗi 5 giây.
- Lệnh tới máy sau **10–20 giây**.

## Dữ liệu trong database

Mọi thứ nằm dưới `/may/<mã bí mật>/`:

| Đường | Ai ghi | Nội dung |
|---|---|---|
| `trangThai` | ESP32 | `today kl pk phien s1 s2 f1 f2 b2n link msg`, `luc` = giờ máy chủ Firebase |
| `theoNgay/<yyyy-mm-dd>` | ESP32 | `[kim loại, phi kim]` của ngày đó |
| `lenh` | trang web | lệnh đang chờ: `DUNG`, `DUNG-XACNHAN`, `XOALOI`. ESP32 lấy xong thì xoá |
| `lenhDaChay` | ESP32 | lệnh vừa lấy và lúc lấy |

**Quy tắc bảo mật** (Firebase console → Realtime Database → Rules): toàn bộ
database bị khoá. Chỉ được đọc/ghi dưới `/may/$ma` khi `$ma` trùng mã bí mật.
Không có mã thì không xem được, không bấm lệnh được.

## Lần đầu mở trang

Bấm **Cài đặt**. Ô địa chỉ database đã điền sẵn, chỉ cần dán **mã bí mật** →
**Lưu & thử**.

- Mã chỉ lưu trong trình duyệt của máy đang mở (localStorage), **không nằm
  trong `index.html`**. Repo công khai nên tuyệt đối không ghi mã vào đây.
- Mỗi điện thoại hoặc máy tính khác phải nhập một lần.
- Dùng máy chung thì bấm **Xoá khỏi máy này** khi xong.

## Nhập cài đặt vào ESP32 (không cần WiFi `PhanLoai-ESP32`)

Cắm cáp USB vào cổng **UART** hoặc cổng **USB** của ESP32, mở Serial Monitor
ở 115200 baud, gõ từng dòng rồi Enter:

```
SHEET URL https://may-phan-loai-default-rtdb.asia-southeast1.firebasedatabase.app
SHEET TOK <mã bí mật>
SHEET ?
```

`SHEET ?` trả về `lan goi cuoi OK (ma 200)` là ESP32 đã gửi lên được.
Tên lệnh vẫn là `SHEET` vì chúng kế thừa từ bản dùng Google Sheet trước đây.

## Trang có gì

| Phần | Nội dung |
|---|---|
| Chấm trạng thái | ESP32 gửi lên cách đây bao lâu. Quá 45 giây thì báo đỏ |
| Trạng thái máy | Bước Board 1 / Board 2, bảng đỏ khi có lỗi |
| Ô đếm | Hôm nay (kim loại / phi kim), từ lúc bật máy, phôi trong khay, tổng 14 ngày |
| Biểu đồ | Sản lượng 14 ngày |
| **DỪNG MÁY** | Tự gửi hai bước `DUNG` rồi `DUNG-XACNHAN` (ESP32 bắt buộc hai bước) |
| **Xoá lỗi** | Gửi `XOALOI`, chỉ bật khi máy đang báo lỗi |
| Huỷ lệnh đang chờ | Hiện khi có lệnh ESP32 chưa lấy |

**Không có nút CHẠY**, cố ý: người bấm từ xa không nhìn thấy máy.

> Nút DỪNG trên trang này phụ thuộc WiFi và Firebase, **không phải nút
> dừng khẩn cấp**. Nút STOP và E-stop trên tủ điện vẫn là thiết bị bảo vệ chính.

## Dò lỗi

| Hiện tượng | Nguyên nhân |
|---|---|
| "Sai mã bí mật" | Mã gõ trên trang khác mã trong quy tắc database |
| "Không gọi được Firebase" | Sai địa chỉ database hoặc mất mạng |
| "ESP32 im lặng từ …" | ESP32 mất điện, mất WiFi, hoặc chưa nhập `SHEET URL` / `SHEET TOK` |
| "Mất tín hiệu Board 1" | ESP32 vẫn lên mạng nhưng không nhận được dữ liệu từ Board 1 (dây D18 → GPIO18, GND) |
