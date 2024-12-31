# Hướng dẫn tạo bot quản lý chi tiêu với Telegram và Google Sheets

---

## I. Tạo bot Telegram
  1.Mở Telegram và tìm BotFather.
 
  2.Gửi lệnh /newbot để tạo bot mới.
 
  3.Đặt tên và username cho bot (username phải kết thúc bằng _bot).
 
  4.Sau khi hoàn tất, BotFather gửi cho bạn Token API 
  Ví dụ: 123456:ABC-DEF1234ghIkl-zyx57W2v1u123ew11.
 
###  → Lưu lại Token API để sử dụng trong Google Apps Script.

---

## II. Tạo Google Sheets và viết Google Apps Script

  1.Tạo một bảng Google Sheets mới, đặt tên (ví dụ: QuanLyChiTieu). 
 
  2.Trong Google Sheets, vào menu Extensions (Tiện ích mở rộng) > Apps Script.
 
  3.Xóa toàn bộ nội dung mặc định và dán mã code:

  
  ```
  
  const TELEGRAM_TOKEN = "YOUR_TELEGRAM_BOT_TOKEN";
const TELEGRAM_API_URL = `https://api.telegram.org/bot${TELEGRAM_TOKEN}`;
const SPREADSHEET_ID = "YOUR_SPREADSHEET_ID";

function doPost(e) {
  const contents = JSON.parse(e.postData.contents);
  if (contents.message) {
    const chatId = contents.message.chat.id;
    const text = contents.message.text;

    if (text.startsWith("/start")) {
      sendMessage(
        chatId,
        "Chào mừng bạn đến với bot quản lý chi tiêu!\n" +
          "Hãy nhập thông tin theo cú pháp: <số tiền> <thu/chi> <mô tả>.\n" +
          "Ví dụ: 500k chi ăn sáng.\n" +
          "Hoặc dùng các lệnh sau:\n" +
          "/report: Báo cáo tổng thu và chi.\n" +
          "/reset: Xóa toàn bộ dữ liệu.\n" +
          "/undo: Xóa giao dịch gần nhất."
      );
    } else if (text.startsWith("/report")) {
      generateReport(chatId);
    } else if (text.startsWith("/reset")) {
      resetReport(chatId);
    } else if (text.startsWith("/undo")) {
      undoLastTransaction(chatId);
    } else {
      const params = text.split(" ");
      if (params.length < 2) {
        sendMessage(chatId, "Lỗi: Nhập đúng cú pháp: <số tiền> <thu/chi> <mô tả>.\nVí dụ: 500k chi ăn sáng.");
      } else {
        const amount = params[0];
        const type = params[1].toLowerCase();
        const description = params.slice(2).join(" ") || "Không có mô tả";
        const date = new Date().toLocaleString();

        if (!isValidAmount(amount)) {
          sendMessage(chatId, "Lỗi: Số tiền không hợp lệ. Hãy nhập lại!");
        } else if (type !== "thu" && type !== "chi") {
          sendMessage(chatId, "Lỗi: Loại giao dịch phải là 'thu' hoặc 'chi'.");
        } else {
          const sheet = SpreadsheetApp.openById(SPREADSHEET_ID).getActiveSheet();
          sheet.appendRow([date, type, amount, description]);
          sendMessage(chatId, `Đã thêm giao dịch:\n- Số tiền: ${amount}\n- Loại: ${type}\n- Mô tả: ${description}`);
        }
      }
    }
  }
}

function isValidAmount(amount) {
  return /^[0-9]+(k|tr)?$/.test(amount);
}

function parseAmount(amount) {
  if (amount.includes("tr")) {
    return parseFloat(amount.replace("tr", "")) * 1000000;
  } else if (amount.includes("k")) {
    return parseFloat(amount.replace("k", "")) * 1000;
  }
  return parseFloat(amount) || 0;
}

function generateReport(chatId) {
  const sheet = SpreadsheetApp.openById(SPREADSHEET_ID).getActiveSheet();
  const data = sheet.getDataRange().getValues();

  let totalIncome = 0;
  let totalExpense = 0;

  for (let i = 0; i < data.length; i++) {
    const row = data[i];
    const type = row[1]?.toLowerCase();
    const amount = parseAmount(row[2]);

    if (type === "thu") {
      totalIncome += amount;
    } else if (type === "chi") {
      totalExpense += amount;
    }
  }

  sendMessage(
    chatId,
    `Báo cáo:\n- Tổng thu: ${formatCurrency(totalIncome)}\n- Tổng chi: ${formatCurrency(totalExpense)}\n- Cân đối: ${formatCurrency(totalIncome - totalExpense)}`
  );
}

function formatCurrency(amount) {
  return new Intl.NumberFormat("vi-VN", { style: "currency", currency: "VND" }).format(amount);
}

function resetReport(chatId) {
  const sheet = SpreadsheetApp.openById(SPREADSHEET_ID).getActiveSheet();
  sheet.clear();
  sendMessage(chatId, "Đã xóa toàn bộ dữ liệu.");
}

function undoLastTransaction(chatId) {
  const sheet = SpreadsheetApp.openById(SPREADSHEET_ID).getActiveSheet();
  const data = sheet.getDataRange().getValues();

  if (data.length > 0) {
    sheet.deleteRow(data.length);
    sendMessage(chatId, "Đã xóa giao dịch gần nhất.");
  } else {
    sendMessage(chatId, "Không có giao dịch nào để xóa.");
  }
}

function sendMessage(chatId, text) {
  const payload = {
    chat_id: chatId,
    text: text,
  };
  const options = {
    method: "post",
    contentType: "application/json",
    payload: JSON.stringify(payload),
  };  UrlFetchApp.fetch(`${TELEGRAM_API_URL}/sendMessage`, options);
}
```

---

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

---

## V. Sử dụng bot
Thêm giao dịch: Gửi tin nhắn với cú pháp: 

      <số tiền> <danh mục> <mô tả>.
 
Ví dụ:
        500k chi ăn sáng.
  ### Các lệnh khác:
  
**/start**: Bắt đầu bot và xem hướng dẫn.
 
**/report**: Báo cáo chi tiêu.
 
**/undo**: Xóa giao dịch gần nhất.
 
**/reset**: Xóa toàn bộ dữ liệu trên Google Sheets.


# Lưu ý

*Google Sheets không được xóa hoặc thay đổi ID.*
 
*Tài khoản Gmail cần cấp quyền cho Google Sheets khi cài Webhook.*
 
*Đảm bảo bot Telegram đã được kết nối đúng Webhook.*

