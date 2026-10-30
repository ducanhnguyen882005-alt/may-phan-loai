# Xem máy phân loại từ xa qua GitHub Pages

`index.html` là trang tĩnh, đưa lên GitHub Pages thì mở được từ bất cứ đâu
(ví dụ `https://<tên-tài-khoản>.github.io/may-phan-loai/`).

## Trang lấy dữ liệu từ đâu

```
ESP32 ──(HTTPS, 30 giây/lần)──► Google Apps Script ◄──(HTTPS)── trang GitHub Pages
         gửi số liệu, lấy lệnh        code/GoogleSheet/Code.gs      action=status / action=cmd
```

Trang trên GitHub **không gọi thẳng ESP32** được: trình duyệt chặn trang
HTTPS gọi vào `http://192.168.x.x`, còn WiFi trường thì chặn mọi kết nối
gọi vào board. Cả ESP32 lẫn trang web cùng gọi ra Apps Script, cách mà
firmware đã dùng sẵn. **Không phải sửa firmware.**

Hệ quả: số liệu trễ tới 30 giây, lệnh tới máy sau 30–60 giây.

## Điều kiện trước

Phần **"Xem máy từ xa qua Google Sheet"** trong `code/HUONG-DAN-ESP32.md`
của đồ án phải chạy được trước:
Apps Script đã triển khai dạng *Ứng dụng web* (truy cập **Bất kỳ ai**), và
đường dẫn cùng mã bí mật đã nhập vào board ở `http://192.168.4.1/sheet`.

## Đưa lên GitHub Pages

1. Vào github.com → **New repository**, đặt tên ví dụ `may-phan-loai`,
   chọn **Public** (GitHub Pages miễn phí chỉ dùng được cho repo công khai).
2. **Add file → Upload files**, kéo `index.html` vào, rồi **Commit**.
3. **Settings → Pages** → *Source*: **Deploy from a branch**, *Branch*:
   **main**, thư mục **/ (root)** → **Save**.
4. Đợi 1–2 phút, trang hiện ở `https://<tên-tài-khoản>.github.io/may-phan-loai/`.

## Lần đầu mở trang

Bấm **Cài đặt**, dán **đường dẫn …/exec** của Apps Script và **mã bí mật**
(`MA_BI_MAT` trong `Code.gs`), rồi **Lưu & thử**.

- Hai thứ này **chỉ lưu trong trình duyệt** của máy đang mở (localStorage),
  **không nằm trong `index.html`**. Repo công khai nên tuyệt đối không ghi
  chúng vào file rồi đẩy lên GitHub.
- Ai không có mã bí mật thì mở trang chỉ thấy ô cài đặt trống, không xem
  được số liệu và không bấm được lệnh.
- Điện thoại khác muốn xem thì nhập lại một lần trên máy đó.
- Dùng máy tính chung thì bấm **Xoá khỏi máy này** khi xong.

## Trang có gì

| Phần | Nội dung |
|---|---|
| Chấm trạng thái | ESP32 gọi lên Google cách đây bao lâu. Quá 90 giây thì báo đỏ |
| Trạng thái máy | Bước Board 1 / Board 2, bảng đỏ khi có lỗi |
| Ô đếm | Hôm nay (kim loại / phi kim), từ lúc bật máy, phôi trong khay, tổng 14 ngày |
| Biểu đồ | Sản lượng 14 ngày |
| **DỪNG MÁY** | Tự gửi hai bước `DUNG` rồi `DUNG-XACNHAN` (ESP32 bắt buộc hai bước) |
| **Xoá lỗi** | Gửi `XOALOI`, chỉ bật khi máy đang báo lỗi |
| Huỷ lệnh đang chờ | Hiện khi có lệnh trong hàng chờ mà ESP32 chưa lấy |

**Không có nút CHẠY**, cố ý: người bấm từ xa không nhìn thấy máy (cùng lý do
đã bỏ nút START trên màn hình GOT). Muốn chạy máy thì bấm nút trên tủ điện.

> Nút DỪNG trên trang này tới máy chậm 30–60 giây và phụ thuộc WiFi lẫn
> Google. Nó **không phải nút dừng khẩn cấp**. Nút STOP và E-stop trên tủ
> điện vẫn là thiết bị bảo vệ chính.

## Dò lỗi

| Hiện tượng | Nguyên nhân |
|---|---|
| "Không gọi được Apps Script" | Sai đường dẫn (phải kết thúc bằng `/exec`), hoặc lúc triển khai chưa chọn *Bất kỳ ai* |
| "Sai mã bí mật" | Mã gõ trên trang khác `MA_BI_MAT` trong `Code.gs` |
| "Board chưa gọi lên lần nào" | ESP32 chưa nhập đường dẫn/mã ở trang `/sheet`, hoặc chưa vào được WiFi có Internet |
| "ESP32 im lặng từ …" | ESP32 mất điện hoặc mất WiFi; số liệu đang hiện là lần cuối |
| Sửa `Code.gs` xong mà trang không đổi | Apps Script phải **Triển khai → Quản lý → sửa → Phiên bản mới**, chỉ lưu thì chưa có hiệu lực |
