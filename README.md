# Hướng dẫn tạo bot quản lý chi tiêu với Telegram và Google Sheets

## I. Tạo bot Telegram
  1.Mở Telegram và tìm BotFather.
 
  2.Gửi lệnh /newbot để tạo bot mới.
 
  3.Đặt tên và username cho bot (username phải kết thúc bằng _bot).
 
  4.Sau khi hoàn tất, BotFather gửi cho bạn Token API (ví dụ: 123456:ABC-DEF1234ghIkl-zyx57W2v1u123ew11).
 
###  → Lưu lại Token API để sử dụng trong Google Apps Script.


## II. Tạo Google Sheets và viết Google Apps Script

  1.Tạo một bảng Google Sheets mới, đặt tên (ví dụ: QuanLyChiTieu). 
 
  2.Trong Google Sheets, vào menu Extensions (Tiện ích mở rộng) > Apps Script.
 
  3.Xóa toàn bộ nội dung mặc định và dán mã code (code đã hướng dẫn trước đó).
 

## IV. Cài đặt Webhook

  1.Trong Apps Script, nhấn Deploy (Triển khai) > New Deployment.
Chọn Web App:

	Description: Nhập mô tả (ví dụ: Telegram Bot).
 
	Execute as: Chọn Me.
 
	Who has access: Chọn Anyone.
 
  2.Nhấn Deploy để tạo URL WebApp (ví dụ:https://script.google.com/macros/s/xxxxxxxxxxxxxxx/exec). Sao chép URL này lại để dùng bước tiếp theo.

 
  3.Cài Webhook cho bot Telegram:
 
    Mở trình duyệt, truy cập:
 
   https://api.telegram.org/botYOUR_TELEGRAM_BOT_TOKEN/setWebhook?url=WEB_APP_URL

(Thay YOUR_TELEGRAM_BOT_TOKEN và WEB_APP_URL(URL vừa lưu khi nãy )bằng thông tin của bạn).

  Nếu thành công, bạn sẽ nhận được thông báo:
 
{"ok":true,"result":true,"description":"Webhook was set"}.

## V. Sử dụng bot
	Thêm giao dịch: Gửi tin nhắn với cú pháp: <số tiền> <danh mục> <mô tả>.
 
     Ví dụ: 500k chi ăn sáng.
  ### Các lệnh khác:
	•/start: Bắt đầu bot và xem hướng dẫn.
 
	•/report: Báo cáo chi tiêu.
 
	•/undo: Xóa giao dịch gần nhất.
 
	•/reset: Xóa toàn bộ dữ liệu trên Google Sheets.


# Lưu ý
	Google Sheets không được xóa hoặc thay đổi ID.
 
	Tài khoản Gmail cần cấp quyền cho Google Sheets khi cài Webhook.
 
	Đảm bảo bot Telegram đã được kết nối đúng Webhook.

