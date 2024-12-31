# Hướng dẫn tạo bot quản lý chi tiêu với Telegram và Google Sheets

---

## I. Tạo bot Telegram
  1.Mở Telegram và tìm BotFather.
 
  2.Gửi lệnh **/newbot** để tạo bot mới.
 
  3.Đặt tên và username cho bot (username phải kết thúc bằng _bot).
 
  4.Sau khi hoàn tất, BotFather gửi cho bạn Token API 
  Ví dụ: 123456:ABC-DEF1234ghIkl-zyx57W2v1u123ew11.
 
  ***Lưu lại Token API để sử dụng trong Google Apps Script*** .

---

## II. Tạo Google Sheets và viết Google Apps Script

  1.Tạo một bảng **Google Sheets** mới, đặt tên (ví dụ: QuanLyChiTieu). 
 
  2.Trong Google Sheets, vào menu **Extensions** (Tiện ích mở rộng) > **Apps Script**.
 

 3.Chuẩn bị **Token mà bạn tạo bot khi nãy được cấp** và **id file google sheet** *(sau ký tự d/ và trước ký tự /edit tại đường dẫn đến file)*

 4.Xóa toàn bộ nội dung mặc định và dán mã code ***(nhớ đổi token bot và id file google sheet)*** :

  
  ```
const TELEGRAM_TOKEN = "TOKEN BOT TELEGRAM CỦA BẠN";
const TELEGRAM_API_URL = `https://api.telegram.org/bot${TELEGRAM_TOKEN}`;
const SPREADSHEET_ID = "ID FILE GOOGLE SHEET CỦA BẠN";

function doPost(e) {
  const contents = JSON.parse(e.postData.contents);
  const chatId = contents.message.chat.id;
  const text = contents.message.text;

  if (text.startsWith("/start")) {
    sendMessage(
      chatId,
      "Chào mừng bạn!\nNhập: <số tiền> <thu/chi> <mô tả>.\nLệnh:\n/report: Báo cáo thu chi\n/reset: Xóa dữ liệu\n/undo: Xóa giao dịch gần nhất."
    );
  } else if (text.startsWith("/report")) generateReport(chatId);
  else if (text.startsWith("/reset")) resetReport(chatId);
  else if (text.startsWith("/undo")) undoLastTransaction(chatId);
  else handleTransaction(chatId, text);
}

function handleTransaction(chatId, text) {
  const [amount, type, ...desc] = text.split(" ");
  if (!isValidAmount(amount) || !["thu", "chi"].includes(type)) {
    sendMessage(chatId, "Lỗi: Nhập đúng cú pháp: <số tiền> <thu/chi> <mô tả>.");
    return;
  }
  const description = desc.join(" ") || "Không có mô tả";
  const sheet = SpreadsheetApp.openById(SPREADSHEET_ID).getActiveSheet();
  sheet.appendRow([new Date().toLocaleString(), type, amount, description]);
  sendMessage(chatId, `Đã thêm giao dịch:\nSố tiền: ${amount}\nLoại: ${type}\nMô tả: ${description}`);
}

function generateReport(chatId) {
  const sheet = SpreadsheetApp.openById(SPREADSHEET_ID).getActiveSheet();
  const data = sheet.getDataRange().getValues();
  
  if (data.length <= 1) {
    sendMessage(chatId, "Không có dữ liệu để báo cáo.");
    return;
  }

  let totalIncome = 0, totalExpense = 0;

  data.slice(1).forEach(row => {
    const type = row[1];
    const amount = parseAmount(row[2]);
    if (type === "thu") totalIncome += amount;
    if (type === "chi") totalExpense += amount;
  });

  const balance = totalIncome - totalExpense;
  sendMessage(
    chatId,
    `Báo cáo thu chi:\nTổng thu: ${formatCurrency(totalIncome)}\nTổng chi: ${formatCurrency(totalExpense)}\nCân đối: ${formatCurrency(balance)}`
  );
}

function resetReport(chatId) {
  const sheet = SpreadsheetApp.openById(SPREADSHEET_ID).getActiveSheet();
  sheet.clear();
  sheet.appendRow(["Thời gian", "Loại", "Số tiền", "Mô tả"]); // Thêm tiêu đề
  sendMessage(chatId, "Đã xóa toàn bộ dữ liệu và đặt lại tiêu đề.");
}

function undoLastTransaction(chatId) {
  const sheet = SpreadsheetApp.openById(SPREADSHEET_ID).getActiveSheet();
  const lastRow = sheet.getLastRow();
  if (lastRow > 1) {
    sheet.deleteRow(lastRow);
    sendMessage(chatId, "Đã xóa giao dịch gần nhất.");
  } else sendMessage(chatId, "Không có giao dịch nào để xóa.");
}

function isValidAmount(amount) {
  return /^[0-9]+(k|tr)?$/.test(amount);
}

function parseAmount(amount) {
  if (amount.includes("tr")) return parseFloat(amount.replace("tr", "")) * 1e6;
  if (amount.includes("k")) return parseFloat(amount.replace("k", "")) * 1e3;
  return parseFloat(amount) || 0;
}

function formatCurrency(amount) {
  return new Intl.NumberFormat("vi-VN", { style: "currency", currency: "VND" }).format(amount);
}

function sendMessage(chatId, text) {
  UrlFetchApp.fetch(`${TELEGRAM_API_URL}/sendMessage`, {
    method: "post",
    contentType: "application/json",
    payload: JSON.stringify({ chat_id: chatId, text }),
  });
}
```

---

## IV. Cài đặt Webhook

1.Trong **Apps Script**, nhấn **Deploy** (Triển khai) > **New Deployment.**
Chọn **Web Application**:

**Description**: Nhập mô tả (ví dụ: Telegram Bot).
 
**Execute as**: Chọn **Me**. 
 
**Who has access**: Chọn **Anyone**(Bắt buộc).

*Xong cấp quyền vào tài khoản gmail của bạn*

2.Nhấn **Deploy** để tạo URL WebApp (ví dụ:https://script.google.com/macros/s/xxxxxxxxxxxxxxx/exec). Sao chép URL này lại để dùng bước tiếp theo.

 
3.Cài **Webhook** cho bot **Telegram:**
  
Mở trình duyệt, truy cập:
                https://api.telegram.org/botYOUR_TELEGRAM_BOT_TOKEN/setWebhook?url=WEB_APP_URL

(Thay **YOUR_TELEGRAM_BOT_TOKEN** và **WEB_APP_URL** *(URL vừa lưu khi nãy )* bằng thông tin của bạn).

  Nếu thành công, bạn sẽ nhận được thông báo:
 
{"ok":true,"result":true,"description":"Webhook was set"}.

---

## V. Sử dụng bot
Thêm giao dịch: Gửi tin nhắn với cú pháp: 

      <số tiền> <danh mục> <mô tả>.
 
Ví dụ:
        500k chi ăn sáng.

        10tr thu tiền lương t10.
  ### Các lệnh khác:
  
**/start**: Bắt đầu bot và xem hướng dẫn.
 
**/report**: Báo cáo chi tiêu.
 
**/undo**: Xóa giao dịch gần nhất.
 
**/reset**: Xóa toàn bộ dữ liệu trên Google Sheets.

***

# Lưu ý

*Quy ước: 1k = 1000VND, 1tr = 1000000VND*

*Không nhập 5tr2 hoặc lẻ, nếu lẻ thì nhập 5200k*

*Google Sheets không được xóa hoặc thay đổi ID.*
 
*Tài khoản Gmail cần cấp quyền cho Google Sheets khi cài Webhook.*
 
*Đảm bảo bot Telegram đã được kết nối đúng Webhook.*

