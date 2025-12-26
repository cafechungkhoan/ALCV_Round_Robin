# 📋 Master Copy Paste Data
# Multi-Source Google Sheets Sync Tool

> **EN:** Sync data from multiple Google Sheets sources to a single destination sheet  
> **VI:** Đồng bộ dữ liệu từ nhiều Google Sheet nguồn về một Sheet đích

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Google Apps Script](https://img.shields.io/badge/Google%20Apps%20Script-Ready-green.svg)](https://developers.google.com/apps-script)

---

## 📖 Table of Contents / Mục Lục

- [Features / Tính năng](#-features--tính-năng)
- [Architecture / Kiến trúc](#-architecture--kiến-trúc)
- [Workflow / Quy trình](#-workflow--quy-trình)
- [Installation / Cài đặt](#-installation--cài-đặt)
- [Configuration / Cấu hình](#-configuration--cấu-hình)
- [Usage / Sử dụng](#-usage--sử-dụng)
- [Troubleshooting / Xử lý lỗi](#-troubleshooting--xử-lý-lỗi)
- [Roadmap / Lộ trình](#-roadmap--lộ-trình)

---

## ✨ Features / Tính năng

### Core Features / Tính năng chính

| Feature | Description (EN) | Mô tả (VI) |
|---------|------------------|------------|
| 🔄 **Multi-Source Sync** | Sync from unlimited Google Sheets | Đồng bộ từ không giới hạn nguồn |
| 📱 **Phone Normalization** | Handle 84xxx, 0xxx, +84 formats | Xử lý nhiều format số điện thoại |
| 🛡️ **Duplicate Protection** | Auto-remove duplicates by phone | Tự động loại bỏ trùng theo SĐT |
| 🔒 **Lock Mechanism** | Prevent concurrent execution | Chống chạy đồng thời |
| ⏰ **Auto Trigger** | Schedule sync (1min/5min/hourly) | Tự động sync theo lịch |
| 🚨 **Emergency Stop** | One-click stop all triggers | Dừng khẩn cấp 1 click |

### Data Protection / Bảo vệ dữ liệu

| Feature | Description (EN) | Mô tả (VI) |
|---------|------------------|------------|
| 📊 **Upsert Logic** | Update existing, insert new | Cập nhật cũ, thêm mới |
| 🔍 **Phone Validation** | Validate phone length (9-15 digits) | Kiểm tra độ dài SĐT |
| 📝 **Debug Logging** | Detailed execution logs | Log chi tiết để debug |
| ⚠️ **Error Handling** | Graceful error recovery | Xử lý lỗi an toàn |

### User Experience / Trải nghiệm người dùng

| Feature | Description (EN) | Mô tả (VI) |
|---------|------------------|------------|
| 📋 **Custom Menu** | Easy-to-use sheet menu | Menu trong Google Sheets |
| 🧹 **Cleanup Tool** | Remove duplicates with one click | Dọn dẹp duplicate 1 click |
| 📈 **Progress Logs** | See sync progress in real-time | Xem tiến trình sync |

---

## 🏗️ Architecture / Kiến trúc

```
┌─────────────────────────────────────────────────────────────────────┐
│                         GOOGLE APPS SCRIPT                          │
│                     (master_copy_paste_data.js)                     │
└─────────────────────────────────────────────────────────────────────┘
                                  │
                    ┌─────────────┴─────────────┐
                    │                           │
                    ▼                           ▼
┌───────────────────────────┐     ┌───────────────────────────┐
│      SOURCE SHEETS        │     │    DESTINATION SHEET      │
│   (Multiple Sources)      │     │   (Single Destination)    │
├───────────────────────────┤     ├───────────────────────────┤
│ ┌─────────────────────┐   │     │                           │
│ │ Source 1: Lead Info │───│─────│──► Phone: 0901234567     │
│ └─────────────────────┘   │     │    Name: Nguyen Van A     │
│ ┌─────────────────────┐   │     │                           │
│ │ Source 2: Ứng viên  │───│─────│──► Phone: 0987654321     │
│ └─────────────────────┘   │     │    Name: Le Thi B         │
│ ┌─────────────────────┐   │     │                           │
│ │ Source N: ...       │───│─────│──► ...                    │
│ └─────────────────────┘   │     │                           │
└───────────────────────────┘     └───────────────────────────┘
```

### Components / Thành phần

| Component | Purpose (EN) | Chức năng (VI) |
|-----------|--------------|----------------|
| `SOURCES` | Array of source configurations | Mảng cấu hình các nguồn |
| `CONFIG` | Destination & protection settings | Cài đặt đích & bảo vệ |
| `syncAllSources()` | Main sync function | Hàm sync chính |
| `syncFromSource_()` | Process single source | Xử lý 1 nguồn |
| `normalizePhone_()` | Standardize phone format | Chuẩn hóa SĐT |

---

## 🔄 Workflow / Quy trình

```
┌──────────────────────────────────────────────────────────────────────┐
│                          SYNC WORKFLOW                                │
└──────────────────────────────────────────────────────────────────────┘

  ┌─────────┐    ┌─────────┐    ┌─────────┐    ┌─────────┐    ┌─────────┐
  │  START  │───►│  LOCK   │───►│  READ   │───►│ COMPARE │───►│  WRITE  │
  │ Trigger │    │ Acquire │    │ Sources │    │ Phones  │    │ Results │
  └─────────┘    └─────────┘    └─────────┘    └─────────┘    └─────────┘
       │              │              │              │              │
       │              │              │              │              │
       ▼              ▼              ▼              ▼              ▼
  ┌─────────┐    ┌─────────┐    ┌─────────┐    ┌─────────┐    ┌─────────┐
  │ Manual  │    │ If busy │    │ Loop    │    │ Exists? │    │ Insert  │
  │ or Auto │    │ → Skip  │    │ sources │    │ Update  │    │ or      │
  │         │    │         │    │         │    │ New?    │    │ Update  │
  └─────────┘    └─────────┘    └─────────┘    └─────────┘    └─────────┘
                                                                    │
                                                                    ▼
                                                              ┌─────────┐
                                                              │ RELEASE │
                                                              │  LOCK   │
                                                              └─────────┘
```

### Step by Step / Từng bước

| Step | Action (EN) | Hành động (VI) |
|------|-------------|----------------|
| 1️⃣ | Trigger starts (manual/auto) | Trigger khởi động |
| 2️⃣ | Acquire lock (skip if busy) | Lấy lock (bỏ qua nếu bận) |
| 3️⃣ | Read existing phones from dest | Đọc phone có sẵn từ đích |
| 4️⃣ | Loop through each source | Lặp qua từng nguồn |
| 5️⃣ | Read source data | Đọc dữ liệu nguồn |
| 6️⃣ | Normalize phones | Chuẩn hóa phone |
| 7️⃣ | Compare & deduplicate | So sánh & loại trùng |
| 8️⃣ | Update existing rows | Cập nhật hàng có sẵn |
| 9️⃣ | Insert new rows | Thêm hàng mới |
| 🔟 | Release lock | Giải phóng lock |

---

## 📥 Installation / Cài đặt

### Prerequisites / Yêu cầu

- Google Account / Tài khoản Google
- Access to source sheets / Quyền truy cập các sheet nguồn
- Edit access to destination sheet / Quyền chỉnh sửa sheet đích

### Steps / Các bước

#### 1. Open Apps Script / Mở Apps Script
```
Google Sheets → Extensions → Apps Script
```

#### 2. Copy Code / Sao chép code
- Delete default code / Xóa code mặc định
- Paste `master_copy_paste_data.js` / Dán code

#### 3. Save / Lưu
- Press `Ctrl+S` or click 💾

#### 4. Authorize / Cấp quyền
- Run `syncAllSources` 
- Click "Review Permissions"
- Allow access

---

## ⚙️ Configuration / Cấu hình

### Sources / Nguồn dữ liệu

```javascript
// Format: ['Name', 'Sheet ID', 'Tab Name', 'Phone Column']
const SOURCES = [
    ['Lead Info',  'YOUR_SHEET_ID_1', 'Sheet1', 'B'],
    ['Candidates', 'YOUR_SHEET_ID_2', 'Data',   'A'],
    // Add more sources here / Thêm nguồn mới ở đây
];
```

> **💡 How to get Sheet ID / Cách lấy Sheet ID:**
> ```
> URL: https://docs.google.com/spreadsheets/d/SHEET_ID_HERE/edit
>                                            ↑↑↑↑↑↑↑↑↑↑↑↑↑
> ```

### Destination / Đích

```javascript
const CONFIG = {
    DESTINATION_SHEET_NAME: 'Sheet1',  // Tab name / Tên tab
    DESTINATION_PHONE_COL: 'B',        // Phone column / Cột phone
    START_ROW: 2,                      // Skip header / Bỏ qua header
    
    // Protection / Bảo vệ
    USE_LOCK: true,
    DEBUG_MODE: true,
    
    // Validation
    MIN_PHONE_LENGTH: 9,
    MAX_PHONE_LENGTH: 15,
};
```

---

## 🎮 Usage / Sử dụng

### Menu Options / Tùy chọn menu

After installation, refresh sheet to see menu:  
Sau khi cài đặt, refresh sheet để thấy menu:

```
📋 Master Sync
├── ▶️ Sync Now / Sync Ngay
├── ⚡ Enable Auto Sync (1 min) / Bật Auto Sync
├── ──────────────
├── 🚨 EMERGENCY STOP / DỪNG KHẨN CẤP
└── 🧹 Remove Duplicates / Xóa Trùng
```

### Functions / Các hàm

| Function | Purpose (EN) | Chức năng (VI) |
|----------|--------------|----------------|
| `syncAllSources()` | Run sync manually | Chạy sync thủ công |
| `setupMinuteTrigger()` | Enable auto sync | Bật auto sync |
| `emergencyStop()` | Stop all triggers | Dừng tất cả trigger |
| `removeDuplicates()` | Clean duplicates | Dọn duplicate |

---

## 🔧 Troubleshooting / Xử lý lỗi

### Common Issues / Lỗi thường gặp

| Issue (EN) | Vấn đề (VI) | Solution / Giải pháp |
|------------|-------------|----------------------|
| Duplicates appearing | Xuất hiện trùng lặp | Run `removeDuplicates()` |
| No access to source | Không có quyền nguồn | Request view access |
| Quota exceeded | Vượt quota | Reduce sync frequency |
| Lock timeout | Timeout lock | Wait and retry |

### Debug Mode / Chế độ Debug

Set `DEBUG_MODE: true` in CONFIG to see detailed logs:  
Đặt `DEBUG_MODE: true` để xem log chi tiết:

```
View → Execution log
```

---

## 🚀 Roadmap / Lộ trình

### Current Version / Phiên bản hiện tại: v1.0.0

### Planned Features / Tính năng dự kiến

| Priority | Feature (EN) | Tính năng (VI) | Status |
|----------|--------------|----------------|--------|
| 🔴 High | Email notifications | Thông báo email | Planned |
| 🔴 High | Column mapping per source | Mapping cột riêng | Planned |
| 🟡 Medium | Webhook integration | Tích hợp webhook | Planned |
| 🟡 Medium | Backup before sync | Backup trước sync | Planned |
| 🟢 Low | Web dashboard | Dashboard web | Future |
| 🟢 Low | Slack notifications | Thông báo Slack | Future |

---

## 📄 License

MIT License - Free to use and modify  
MIT License - Tự do sử dụng và chỉnh sửa

---

## 🤝 Contributing / Đóng góp

Pull requests are welcome!  
Chào đón mọi đóng góp!

---

## 📞 Support / Hỗ trợ

- GitHub Issues: [Create an issue](../../issues)
- Email: your-email@example.com

---

Made with ❤️ for productivity
