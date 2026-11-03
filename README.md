# Xem máy phân loại từ xa

`index.html` là trang tĩnh, mở được từ bất cứ đâu:
**https://may-phan-loai.web.app** (Firebase Hosting của dự án `may-phan-loai`).

Đưa bản mới lên (chạy trong thư mục đồ án, đã `firebase login` một lần):

```
firebase deploy --only hosting --project may-phan-loai --config "code/GitHubPages/firebase.json"
```

`firebase.json` chặn không cho `README.md`, `.git` và các file cấu hình lên trang.

## Trang lấy dữ liệu từ đâu

```
ESP32 (WiFi trường) ──HTTPS, giữ kết nối──► Firebase Realtime Database ──kênh đẩy (SSE)──► trang GitHub Pages
                                             (gói Spark, miễn phí)
```

Trang **không gọi thẳng ESP32**: trình duyệt chặn trang HTTPS gọi vào
`http://192.168.x.x`, còn WiFi trường thì chặn mọi kết nối gọi vào board.
Cả ESP32 lẫn trang web cùng gọi ra Firebase.

- ESP32 gửi **ngay khi số liệu đổi** (kiểm tra 0,2 s/lần, mỗi lần gửi
  ~1,5 s khi máy chạy). Máy đứng yên thì 5 s gửi một nhịp.
- Trang **không hỏi lặp lại**: Firebase đẩy thay đổi xuống qua
  EventSource, hiện gần như tức thì.
- ESP32 hỏi lệnh mỗi giây: lệnh tới ESP32 sau **0,5–1,5 giây** (đo ngày
  03/11/2026). Lệnh DỪNG hai bước mất khoảng 2–5 giây.

## Dữ liệu trong database

Mọi thứ nằm dưới `/may/<mã bí mật>/`:

| Đường | Ai ghi | Nội dung |
|---|---|---|
| `trangThai` | ESP32 | `today kl pk kr phien s1 s2 f1 f2 b2n link msg in1 out in2 cell m3 m4`, `luc` = giờ máy chủ Firebase |
| `theoNgay/<yyyy-mm-dd>` | ESP32 | `[kim loại, phi kim, chưa rõ loại]` của ngày đó |
| `nhatKy` | ESP32 | 40 dòng nhật ký gần nhất, mới nhất ở đầu: `{t, w, m}` |
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

Cắm cáp USB vào cổng **USB** của ESP32 (không phải cổng UART), mở Serial Monitor
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
| Chấm trạng thái | ESP32 gửi lên cách đây bao lâu. Quá 20 giây thì báo đỏ |
| Trạng thái máy | Bước Board 1 / Board 2, bảng đỏ khi có lỗi |
| Ô đếm | Hôm nay (kim loại / phi kim), từ lúc bật máy, phôi trong khay, tổng 14 ngày |
| Biểu đồ | Sản lượng 14 ngày |
| Cảm biến | Đèn từng cảm biến Board 1 / Board 2, van và động cơ, vị trí vít me |
| Nhật ký | 40 dòng gần nhất: lỗi, cảnh báo, các lần dừng / xoá lỗi, phôi cộng bù |
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
