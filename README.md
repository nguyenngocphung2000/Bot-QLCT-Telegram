# Hướng dẫn tạo bot quản lý chi tiêu với Telegram và Google Sheets

---

## Mục tiêu của bot

   Bot không chỉ là một công cụ giúp ghi chép mà còn hướng đến việc xây dựng thói quen tài chính tốt hơn cho người dùng. Với tính năng báo cáo và phân tích, bot mang lại sự minh bạch và kiểm soát chặt chẽ, giúp người dùng quản lý chi tiêu thông minh hơn và đạt được mục tiêu tài chính cá nhân.
   
***

## Tác dụng của bot

  *Bot được thiết kế như một công cụ quản lý tài chính cá nhân đơn giản nhưng hiệu quả. Nó giúp người dùng ghi lại, theo dõi, và báo cáo các giao dịch thu nhập và chi tiêu trong cuộc sống hàng ngày. Dưới đây là một số lợi ích chính*

# 1.
Quản lý tài chính cá nhân:
Người dùng có thể ghi chép các khoản thu nhập và chi tiêu một cách nhanh chóng, dễ dàng qua Telegram.

# 2.
Theo dõi chi tiết giao dịch:
Bot cung cấp báo cáo chi tiết về các giao dịch trong một khoảng thời gian cụ thể, giúp người dùng có cái nhìn tổng quan về tình hình tài chính.

# 3.
Hỗ trợ phân tích tài chính:
Với các báo cáo cân đối giữa thu và chi, người dùng có thể xác định các vấn đề như chi tiêu quá mức hoặc nguồn thu nhập chính yếu của mình.

# 4.
Tăng tính minh bạch và tổ chức:
Bot ghi lại mọi giao dịch vào Google Sheets, giúp lưu trữ dữ liệu an toàn và dễ dàng truy cập để xem lại.

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
const TOKEN = "TOKEN BOT TELEGRAM CỦA BẠN";
const API_URL = `https://api.telegram.org/bot${TOKEN}`;
const SHEET_ID = "ID FILE SHEET CỦA BẠN";

function doPost(e) {
  const { message } = JSON.parse(e.postData.contents);
  const chatId = message.chat.id;
  const text = message.text;

  if (text.startsWith("/start")) {
    sendMessage(chatId, `Chào mừng!\nNhập: <số tiền> <thu/chi> <mô tả>.\nLệnh:\n/report: Tổng\n/report month <MM-YYYY>\n/report week <DD-MM-YYYY>\n/reset: Xóa dữ liệu\n/undo: Xóa giao dịch gần nhất.`);
  } else if (text.startsWith("/report")) handleReport(chatId, text);
  else if (text.startsWith("/reset")) resetSheet(chatId);
  else if (text.startsWith("/undo")) undoLast(chatId);
  else handleTransaction(chatId, text);
}

function handleTransaction(chatId, text) {
  const [amount, type, ...desc] = text.split(" ");
  if (!isValidAmount(amount) || !["thu", "chi"].includes(type)) {
    sendMessage(chatId, "Lỗi: Nhập đúng cú pháp <số tiền> <thu/chi> <mô tả>.");
    return;
  }

  const sheet = SpreadsheetApp.openById(SHEET_ID).getActiveSheet();
  sheet.appendRow([new Date(), type, parseAmount(amount), desc.join(" ") || "Không có mô tả"]);
  sendMessage(chatId, `Đã thêm giao dịch:\nSố tiền: ${amount}\nLoại: ${type}\nMô tả: ${desc.join(" ")}`);
}

function handleReport(chatId, text) {
  const filter = text.includes("month") ? "month" : text.includes("week") ? "week" : "all";
  const dateParam = text.match(/\d{2}-\d{4}|\d{2}-\d{2}-\d{4}/)?.[0] || null;
  generateReport(chatId, filter, dateParam);
}

function generateReport(chatId, filter, dateParam) {
  const sheet = SpreadsheetApp.openById(SHEET_ID).getActiveSheet();
  const data = sheet.getDataRange().getValues().slice(1); // Tải toàn bộ dữ liệu một lần
  if (!data.length) {
    sendMessage(chatId, "Không có dữ liệu.");
    return;
  }

  const now = parseDate(filter, dateParam);
  const filteredData = data.filter(([date]) => isValidDate(new Date(date), filter, now));
  
  const incomeTransactions = [];
  const expenseTransactions = [];
  let [income, expense] = [0, 0];

  filteredData.forEach(([date, type, amount, desc]) => {
    const formattedDate = new Date(date).toLocaleDateString("vi-VN");
    const transaction = `${formatCurrency(amount)}: ${desc || "Không có mô tả"} (${formattedDate})`;

    if (type === "thu") {
      income += amount;
      incomeTransactions.push(`+ ${transaction}`);
    } else if (type === "chi") {
      expense += amount;
      expenseTransactions.push(`- ${transaction}`);
    }
  });

  const report = [
    `Báo cáo (${filter === "all" ? "tổng" : filter}):`,
    `Tổng thu: ${formatCurrency(income)}`,
    `Tổng chi: ${formatCurrency(expense)}`,
    `Cân đối: ${formatCurrency(income - expense)}`,
    "",
    "Giao dịch thu nhập cụ thể:",
    incomeTransactions.length ? incomeTransactions.join("\n") : "Không có giao dịch thu nhập.",
    "",
    "Giao dịch chi tiêu cụ thể:",
    expenseTransactions.length ? expenseTransactions.join("\n") : "Không có giao dịch chi tiêu.",
  ].join("\n");

  sendMessage(chatId, report);
}

function resetSheet(chatId) {
  const sheet = SpreadsheetApp.openById(SHEET_ID).getActiveSheet();
  sheet.clear();
  sheet.appendRow(["Thời gian", "Loại", "Số tiền", "Mô tả"]); // Thêm tiêu đề sau khi xóa
  sendMessage(chatId, "Đã xóa toàn bộ dữ liệu.");
}

function undoLast(chatId) {
  const sheet = SpreadsheetApp.openById(SHEET_ID).getActiveSheet();
  const lastRow = sheet.getLastRow();
  if (lastRow > 1) {
    sheet.deleteRow(lastRow);
    sendMessage(chatId, "Đã xóa giao dịch gần nhất.");
  } else {
    sendMessage(chatId, "Không có giao dịch nào để xóa.");
  }
}

function isValidDate(date, filter, now) {
  if (filter === "month") {
    return date.getMonth() === now.getMonth() && date.getFullYear() === now.getFullYear();
  }
  if (filter === "week") {
    const startOfWeek = new Date(now);
    startOfWeek.setDate(now.getDate() - now.getDay());
    const endOfWeek = new Date(startOfWeek);
    endOfWeek.setDate(startOfWeek.getDate() + 6);
    return date >= startOfWeek && date <= endOfWeek;
  }
  return true;
}

function parseDate(filter, dateParam) {
  if (!dateParam) return new Date();
  const parts = dateParam.split("-");
  if (filter === "month") return new Date(parts[1], parts[0] - 1);
  if (filter === "week") return new Date(parts[2], parts[1] - 1, parts[0]);
}

function isValidAmount(amount) {
  return /^[0-9]+(k|tr)?$/.test(amount);
}

function parseAmount(amount) {
  return parseFloat(amount.replace("tr", "000000").replace("k", "000")) || 0;
}

function formatCurrency(amount) {
  return new Intl.NumberFormat("vi-VN", { style: "currency", currency: "VND" }).format(amount);
}

function sendMessage(chatId, text) {
  UrlFetchApp.fetch(`${API_URL}/sendMessage`, {
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

![](https://github.com/nguyenngocphung2000/Bot-QLCT-Telegram/blob/main/Thu.PNG)       

![](https://github.com/nguyenngocphung2000/Bot-QLCT-Telegram/blob/main/Chi.PNG)


        
   ***Các lệnh khác:***
  
**/start**: Bắt đầu bot và xem hướng dẫn.
 
**/report**: Báo cáo chi tiêu.

![](https://github.com/nguyenngocphung2000/Bot-QLCT-Telegram/blob/main/Rp.PNG)


**/report month MM-YYYY :** Báo cáo theo tháng.

![](https://github.com/nguyenngocphung2000/Bot-QLCT-Telegram/blob/main/Rp_month.PNG)

**/report week DD-MM-YYYY :** Báo cáo theo tuần có ngày đó

 ![](https://github.com/nguyenngocphung2000/Bot-QLCT-Telegram/blob/main/Rp_week.PNG)
 
**/undo**: Xóa giao dịch gần nhất.
 
**/reset**: Xóa toàn bộ dữ liệu trên Google Sheets.

***

# Lưu ý

*Quy ước: 1k = 1000VND, 1tr = 1000000VND*

*Không nhập 5tr2 hoặc lẻ, nếu lẻ thì nhập 5200k*

*Google Sheets không được xóa hoặc thay đổi ID.*
 
*Tài khoản Gmail cần cấp quyền cho Google Sheets khi cài Webhook.*
 
*Đảm bảo bot Telegram đã được kết nối đúng Webhook.*

