# Reporting & Analytics
# Báo cáo & Phân tích

## Overview (Tổng quan)

Reporting & Analytics giúp:
- Theo dõi hoạt động phòng khám
- Đánh giá hiệu quả điều trị
- Quản lý tài chính
- Tuân thủ quy định
- Ra quyết định dựa trên dữ liệu

**Tại sao quan trọng?**
- Quản lý cần số liệu để ra quyết định
- Bác sĩ cần theo dõi kết quả điều trị
- Kế toán cần báo cáo tài chính
- Tuân thủ quy định bắt buộc

---

## 1. Types of Reports (Các loại báo cáo)

### 1.1 Clinical Reports (Báo cáo lâm sàng)

| Report Type | Mô tả | Người dùng | Tần suất |
|-------------|-------|------------|----------|
| **Patient Summary** | Tóm tắt hồ sơ bệnh nhân | Bác sĩ | On-demand |
| **Diagnosis Report** | Thống kê chẩn đoán | Bác sĩ, Quản lý | Hàng ngày/Tuần |
| **Treatment Outcomes** | Kết quả điều trị | Bác sĩ | Hàng tháng |
| **Lab Results Summary** | Tổng hợp kết quả xét nghiệm | Bác sĩ | Hàng ngày |
| **Prescription Report** | Thống kê đơn thuốc | Bác sĩ, Dược sĩ | Hàng tuần |
| **Vital Signs Trend** | Xu hướng vital signs | Bác sĩ | On-demand |

### 1.2 Operational Reports (Báo cáo vận hành)

| Report Type | Mô tả | Người dùng | Tần suất |
|-------------|-------|------------|----------|
| **Daily Activity** | Hoạt động trong ngày | Quản lý | Hàng ngày |
| **Appointment Statistics** | Thống kê lịch hẹn | Quản lý | Hàng ngày/Tuần |
| **Patient Flow** | Luồng bệnh nhân | Quản lý | Hàng ngày |
| **Doctor Productivity** | Năng suất bác sĩ | Quản lý | Hàng tuần/Tháng |
| **No-show Rate** | Tỷ lệ không đến | Quản lý | Hàng tuần |
| **Queue Management** | Quản lý hàng đợi | Quản lý | Real-time |

### 1.3 Financial Reports (Báo cáo tài chính)

| Report Type | Mô tả | Người dùng | Tần suất |
|-------------|-------|------------|----------|
| **Daily Revenue** | Doanh thu ngày | Kế toán, Quản lý | Hàng ngày |
| **Monthly Revenue** | Doanh thu tháng | Kế toán, Quản lý | Hàng tháng |
| **Payment Summary** | Tổng hợp thanh toán | Kế toán | Hàng ngày |
| **BHYT Claims** | Báo cáo BHYT | Kế toán | Hàng tuần/Tháng |
| **Outstanding Bills** | Công nợ | Kế toán | Hàng ngày |
| **Service Revenue** | Doanh thu theo dịch vụ | Quản lý | Hàng tháng |
| **Profit & Loss** | Báo cáo lãi lỗ | Quản lý | Hàng tháng |

### 1.4 Administrative Reports (Báo cáo hành chính)

| Report Type | Mô tả | Người dùng | Tần suất |
|-------------|-------|------------|----------|
| **Patient Registration** | Đăng ký bệnh nhân | Quản lý | Hàng ngày/Tuần |
| **User Activity** | Hoạt động người dùng | IT Admin | Hàng tuần |
| **Audit Trail** | Nhật ký truy cập | IT Admin, Quản lý | On-demand |
| **Data Export** | Xuất dữ liệu | Quản lý | On-demand |

---

## 2. Clinical Reports

### 2.1 Patient Summary Report

**Purpose:** Tóm tắt toàn bộ hồ sơ bệnh nhân

**Content:**
```
┌─────────────────────────────────────────────────┐
│  TÓM TẮT HỒ SƠ BỆNH NHÂN                       │
│  Mã BN: P2023001234                             │
│  Họ tên: Nguyễn Văn A                          │
│  Ngày sinh: 15/05/1980                          │
└─────────────────────────────────────────────────┘

THÔNG TIN CƠ BẢN:
- Tuổi: 43
- Giới tính: Nam
- BHYT: Có (Số thẻ: 1234567890123)
- Địa chỉ: 123 Lê Loi, Q1, TPHCM

LỊCH SỬ KHÁM BỆNH:
┌─────────────┬──────────────┬──────────────────┐
│ Ngày        │ Bác sĩ       │ Chẩn đoán        │
├─────────────┼──────────────┼──────────────────┤
│ 19/11/2023 │ Dr. Nguyễn B │ Tăng huyết áp    │
│ 15/10/2023 │ Dr. Nguyễn B │ Tăng huyết áp    │
│ 10/09/2023 │ Dr. Trần C   │ Cảm cúm          │
└─────────────┴──────────────┴──────────────────┘

ĐƠN THUỐC GẦN ĐÂY:
- 19/11/2023: Amlodipine 5mg, 1 viên/ngày
- 15/10/2023: Amlodipine 5mg, 1 viên/ngày

XÉT NGHIỆM GẦN ĐÂY:
- 19/11/2023: CBC, Lipid Profile
- 15/10/2023: Blood Pressure Monitoring

DỊ ỨNG:
- Penicillin

BỆNH NỀN:
- Tăng huyết áp (từ 10/2023)
```

### 2.2 Diagnosis Statistics Report

**Purpose:** Thống kê các chẩn đoán phổ biến

**Content:**
```
┌─────────────────────────────────────────────────┐
│  THỐNG KÊ CHẨN ĐOÁN - Tháng 11/2023            │
└─────────────────────────────────────────────────┘

TOP 10 CHẨN ĐOÁN:
┌─────────────────────────────┬────────┬──────────┐
│ Chẩn đoán                   │ Số BN  │ Tỷ lệ   │
├─────────────────────────────┼────────┼──────────┤
│ Tăng huyết áp (I10)         │ 245    │ 18.5%   │
│ Đái tháo đường (E11)        │ 189    │ 14.3%   │
│ Cảm cúm (J11)               │ 156    │ 11.8%   │
│ Viêm phế quản (J40)         │ 134    │ 10.1%   │
│ Đau đầu (R51)               │ 98     │ 7.4%    │
│ Viêm dạ dày (K29)           │ 87     │ 6.6%    │
│ Viêm khớp (M25)             │ 76     │ 5.7%    │
│ Rối loạn tiêu hóa (K59)     │ 65     │ 4.9%    │
│ Viêm họng (J02)             │ 54     │ 4.1%    │
│ Đau lưng (M54)              │ 45     │ 3.4%    │
└─────────────────────────────┴────────┴──────────┘

Tổng số bệnh nhân: 1,320
Tổng số chẩn đoán: 1,325 (một số BN có nhiều chẩn đoán)
```

### 2.3 Treatment Outcomes Report

**Purpose:** Đánh giá hiệu quả điều trị

**Content:**
```
┌─────────────────────────────────────────────────┐
│  KẾT QUẢ ĐIỀU TRỊ - Tăng huyết áp              │
│  Thời gian: 01/10/2023 - 30/11/2023            │
└─────────────────────────────────────────────────┘

TỔNG QUAN:
- Số bệnh nhân điều trị: 245
- Số bệnh nhân tái khám: 189 (77.1%)
- Số bệnh nhân không tái khám: 56 (22.9%)

KẾT QUẢ ĐIỀU TRỊ:
┌─────────────────────────────┬────────┬──────────┐
│ Kết quả                     │ Số BN  │ Tỷ lệ   │
├─────────────────────────────┼────────┼──────────┤
│ Kiểm soát tốt               │ 142    │ 75.1%   │
│ Cải thiện                   │ 32     │ 16.9%   │
│ Không cải thiện             │ 10     │ 5.3%    │
│ Chưa đánh giá               │ 5      │ 2.6%    │
└─────────────────────────────┴────────┴──────────┘

THUỐC ĐƯỢC SỬ DỤNG:
- Amlodipine: 156 BN (82.5%)
- Losartan: 45 BN (23.8%)
- Enalapril: 28 BN (14.8%)
```

---

## 3. Operational Reports

### 3.1 Daily Activity Report

**Purpose:** Tổng hợp hoạt động trong ngày

**Content:**
```
┌─────────────────────────────────────────────────┐
│  BÁO CÁO HOẠT ĐỘNG NGÀY - 19/11/2023           │
└─────────────────────────────────────────────────┘

BỆNH NHÂN:
- Đăng ký mới: 23
- Tổng số khám: 156
- Tái khám: 133 (85.3%)
- Khám mới: 23 (14.7%)

LỊCH HẸN:
- Tổng số lịch hẹn: 180
- Đã đến: 145 (80.6%)
- Không đến (No-show): 15 (8.3%)
- Đã hủy: 20 (11.1%)

BÁC SĨ:
┌─────────────────────┬──────────┬──────────────┐
│ Bác sĩ              │ Số BN    │ Thời gian TB │
├─────────────────────┼──────────┼──────────────┤
│ Dr. Nguyễn Văn A    │ 45       │ 18 phút      │
│ Dr. Trần Thị B      │ 38       │ 20 phút      │
│ Dr. Lê Văn C        │ 35       │ 22 phút      │
│ Dr. Phạm Thị D      │ 38       │ 19 phút      │
└─────────────────────┴──────────┴──────────────┘

DỊCH VỤ:
- Khám bệnh: 156
- Xét nghiệm: 89
- Kê đơn: 142
- Cấp phát thuốc: 138
```

### 3.2 Appointment Statistics Report

**Purpose:** Thống kê lịch hẹn

**Content:**
```
┌─────────────────────────────────────────────────┐
│  THỐNG KÊ LỊCH HẸN - Tuần 46/2023              │
└─────────────────────────────────────────────────┘

TỔNG QUAN:
- Tổng số lịch hẹn: 1,245
- Đã đến: 1,012 (81.3%)
- Không đến: 98 (7.9%)
- Đã hủy: 135 (10.8%)

THEO KÊNH ĐẶT LỊCH:
┌─────────────┬────────┬──────────┐
│ Kênh        │ Số lượng│ Tỷ lệ    │
├─────────────┼────────┼──────────┤
│ Online      │ 567    │ 45.5%    │
│ Phone       │ 423    │ 34.0%    │
│ Walk-in     │ 189    │ 15.2%    │
│ In-person   │ 66     │ 5.3%     │
└─────────────┴────────┴──────────┘

THEO LOẠI KHÁM:
┌─────────────┬────────┬──────────┐
│ Loại        │ Số lượng│ Tỷ lệ    │
├─────────────┼────────┼──────────┤
│ Tái khám     │ 856    │ 68.8%    │
│ Khám mới     │ 312    │ 25.1%    │
│ Khám định kỳ │ 77     │ 6.2%     │
└─────────────┴────────┴──────────┘

NO-SHOW RATE THEO BÁC SĨ:
┌─────────────────────┬──────────┬──────────────┐
│ Bác sĩ              │ No-show  │ Tỷ lệ        │
├─────────────────────┼──────────┼──────────────┤
│ Dr. Nguyễn Văn A    │ 12       │ 6.7%         │
│ Dr. Trần Thị B      │ 8        │ 5.2%         │
│ Dr. Lê Văn C        │ 15       │ 9.8%         │
│ Dr. Phạm Thị D      │ 10       │ 6.5%         │
└─────────────────────┴──────────┴──────────────┘
```

### 3.3 Doctor Productivity Report

**Purpose:** Đánh giá năng suất bác sĩ

**Content:**
```
┌─────────────────────────────────────────────────┐
│  NĂNG SUẤT BÁC SĨ - Tháng 11/2023              │
└─────────────────────────────────────────────────┘

┌─────────────────────┬──────────┬──────────┬──────────────┐
│ Bác sĩ              │ Số BN    │ Giờ TB   │ Đánh giá     │
├─────────────────────┼──────────┼──────────┼──────────────┤
│ Dr. Nguyễn Văn A    │ 1,245    │ 18 phút  │ ⭐⭐⭐⭐⭐    │
│ Dr. Trần Thị B      │ 1,189    │ 20 phút  │ ⭐⭐⭐⭐      │
│ Dr. Lê Văn C        │ 1,156    │ 22 phút  │ ⭐⭐⭐⭐      │
│ Dr. Phạm Thị D      │ 1,198    │ 19 phút  │ ⭐⭐⭐⭐⭐    │
└─────────────────────┴──────────┴──────────┴──────────────┘

CHI TIẾT THEO NGÀY:
┌─────────────┬──────────┬──────────┬──────────┬──────────┐
│ Ngày        │ Dr. A    │ Dr. B    │ Dr. C    │ Dr. D    │
├─────────────┼──────────┼──────────┼──────────┼──────────┤
│ 01/11       │ 45       │ 42       │ 38       │ 44       │
│ 02/11       │ 48       │ 45       │ 40       │ 46       │
│ ...         │ ...      │ ...      │ ...      │ ...      │
│ 30/11       │ 46       │ 43       │ 39       │ 45       │
└─────────────┴──────────┴──────────┴──────────┴──────────┘
```

---

## 4. Financial Reports

### 4.1 Daily Revenue Report

**Purpose:** Doanh thu trong ngày

**Content:**
```
┌─────────────────────────────────────────────────┐
│  BÁO CÁO DOANH THU NGÀY - 19/11/2023           │
└─────────────────────────────────────────────────┘

TỔNG DOANH THU: 125,450,000 VNĐ

THEO LOẠI THANH TOÁN:
┌─────────────────────┬──────────────┬──────────┐
│ Loại                │ Số tiền      │ Tỷ lệ    │
├─────────────────────┼──────────────┼──────────┤
│ Tiền mặt            │ 45,200,000   │ 36.1%    │
│ Chuyển khoản        │ 38,500,000   │ 30.7%    │
│ BHYT                │ 35,750,000   │ 28.5%    │
│ Thẻ tín dụng        │ 6,000,000    │ 4.8%     │
└─────────────────────┴──────────────┴──────────┘

THEO DỊCH VỤ:
┌─────────────────────┬──────────────┬──────────┐
│ Dịch vụ             │ Số tiền      │ Tỷ lệ    │
├─────────────────────┼──────────────┼──────────┤
│ Khám bệnh           │ 31,200,000   │ 24.9%    │
│ Thuốc               │ 58,500,000   │ 46.7%    │
│ Xét nghiệm          │ 22,350,000   │ 17.8%    │
│ Chẩn đoán hình ảnh  │ 8,900,000    │ 7.1%     │
│ Dịch vụ khác        │ 4,500,000    │ 3.6%     │
└─────────────────────┴──────────────┴──────────┘

SỐ HÓA ĐƠN:
- Tổng số hóa đơn: 156
- Hóa đơn BHYT: 89 (57.1%)
- Hóa đơn tự chi trả: 67 (42.9%)
- Hóa đơn chưa thanh toán: 12 (7.7%)
```

### 4.2 Monthly Revenue Report

**Purpose:** Doanh thu tháng

**Content:**
```
┌─────────────────────────────────────────────────┐
│  BÁO CÁO DOANH THU THÁNG - 11/2023              │
└─────────────────────────────────────────────────┘

TỔNG DOANH THU: 3,245,600,000 VNĐ

SO SÁNH THÁNG TRƯỚC:
- Tháng 10/2023: 3,189,500,000 VNĐ
- Tháng 11/2023: 3,245,600,000 VNĐ
- Tăng trưởng: +56,100,000 VNĐ (+1.8%)

THEO NGÀY:
┌─────────────┬──────────────┬──────────────┐
│ Ngày        │ Doanh thu    │ So với TB    │
├─────────────┼──────────────┼──────────────┤
│ 01/11       │ 98,500,000   │ -8.2%        │
│ 02/11       │ 105,200,000  │ -1.9%        │
│ ...         │ ...          │ ...          │
│ 19/11       │ 125,450,000  │ +18.5%       │
│ ...         │ ...          │ ...          │
│ 30/11       │ 112,300,000  │ +6.1%        │
└─────────────┴──────────────┴──────────────┘

THEO LOẠI THANH TOÁN:
┌─────────────────────┬──────────────┬──────────┐
│ Loại                │ Số tiền      │ Tỷ lệ    │
├─────────────────────┼──────────────┼──────────┤
│ Tiền mặt            │ 1,189,200,000│ 36.6%    │
│ Chuyển khoản        │ 995,800,000  │ 30.7%    │
│ BHYT                │ 925,200,000 │ 28.5%    │
│ Thẻ tín dụng        │ 135,400,000  │ 4.2%     │
└─────────────────────┴──────────────┴──────────┘
```

### 4.3 BHYT Claims Report

**Purpose:** Báo cáo BHYT claims

**Content:**
```
┌─────────────────────────────────────────────────┐
│  BÁO CÁO BHYT CLAIMS - Tháng 11/2023            │
└─────────────────────────────────────────────────┘

TỔNG QUAN:
- Tổng số claims: 2,456
- Tổng giá trị: 925,200,000 VNĐ
- Đã submit: 2,456 (100%)
- Đã thanh toán: 2,189 (89.1%)
- Đang chờ: 267 (10.9%)

TRẠNG THÁI CLAIMS:
┌─────────────────────┬──────────┬──────────────┐
│ Trạng thái          │ Số lượng │ Giá trị      │
├─────────────────────┼──────────┼──────────────┤
│ Đã thanh toán        │ 2,189    │ 825,100,000  │
│ Đang chờ thanh toán  │ 267      │ 100,100,000  │
└─────────────────────┴──────────┴──────────────┘

THEO LOẠI DỊCH VỤ:
┌─────────────────────┬──────────┬──────────────┐
│ Dịch vụ             │ Số lượng │ Giá trị      │
├─────────────────────┼──────────┼──────────────┤
│ Khám bệnh           │ 2,456    │ 245,600,000  │
│ Thuốc               │ 1,856    │ 445,200,000  │
│ Xét nghiệm          │ 1,234    │ 185,400,000  │
│ Chẩn đoán hình ảnh  │ 456      │ 49,000,000   │
└─────────────────────┴──────────┴──────────────┘
```

---

## 5. Report Generation Flow

### 5.1 Report Request Flow

```
User requests report
    ↓
Select report type
    ↓
Set parameters:
- Date range
- Filters (doctor, department, etc.)
- Format (PDF, Excel, CSV)
    ↓
System generates report
    ↓
  Real-time report:
  → Query database
  → Format data
  → Display/Download
  
  Scheduled report:
  → Queue job
  → Generate in background
  → Send via email
  → Store in report archive
```

### 5.2 Report Scheduling

**Scheduled Reports:**
- Daily: Daily Activity, Daily Revenue
- Weekly: Appointment Statistics, BHYT Claims
- Monthly: Monthly Revenue, Doctor Productivity, Treatment Outcomes

**Schedule Configuration:**
```
Report: Daily Revenue Report
Schedule: Every day at 18:00
Recipients: clinic_manager@example.com, accountant@example.com
Format: PDF + Excel
```

---

## 6. Data Model for Reports

### 6.1 Report Definitions

```sql
CREATE TABLE report_definitions (
  report_id INT PRIMARY KEY AUTO_INCREMENT,
  report_code VARCHAR(50) UNIQUE NOT NULL,      -- DAILY_REVENUE, PATIENT_SUMMARY...
  report_name_vi VARCHAR(255) NOT NULL,
  report_name_en VARCHAR(255),
  
  -- Report type
  report_category ENUM('Clinical', 'Operational', 'Financial', 'Administrative'),
  
  -- Configuration
  query_template TEXT,                          -- SQL template hoặc query config
  parameters JSON,                              -- Required parameters
  
  -- Access control
  allowed_roles JSON,                           -- Roles có thể xem report này
  
  -- Scheduling
  can_schedule BOOLEAN DEFAULT TRUE,
  default_schedule VARCHAR(50),                 -- DAILY, WEEKLY, MONTHLY
  
  is_active BOOLEAN DEFAULT TRUE,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### 6.2 Report Executions

```sql
CREATE TABLE report_executions (
  execution_id INT PRIMARY KEY AUTO_INCREMENT,
  report_id INT NOT NULL,
  user_id INT,                                  -- User requested report
  
  -- Parameters used
  parameters JSON,                              -- Date range, filters, etc.
  
  -- Execution details
  status ENUM('Pending', 'Running', 'Completed', 'Failed') DEFAULT 'Pending',
  started_at TIMESTAMP,
  completed_at TIMESTAMP,
  execution_time_ms INT,                        -- Thời gian chạy (milliseconds)
  
  -- Output
  file_path VARCHAR(500),                       -- Path to generated file
  file_format VARCHAR(20),                     -- PDF, Excel, CSV
  file_size_bytes INT,
  
  -- Error handling
  error_message TEXT,
  
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  
  FOREIGN KEY (report_id) REFERENCES report_definitions(report_id),
  FOREIGN KEY (user_id) REFERENCES users(user_id),
  
  INDEX idx_report (report_id),
  INDEX idx_user (user_id),
  INDEX idx_status (status),
  INDEX idx_created_at (created_at)
);
```

### 6.3 Report Schedules

```sql
CREATE TABLE report_schedules (
  schedule_id INT PRIMARY KEY AUTO_INCREMENT,
  report_id INT NOT NULL,
  
  -- Schedule config
  frequency ENUM('Daily', 'Weekly', 'Monthly') NOT NULL,
  schedule_time TIME,                            -- Time to run (VD: 18:00)
  day_of_week TINYINT,                          -- For weekly (1=Monday)
  day_of_month TINYINT,                         -- For monthly (1-31)
  
  -- Recipients
  recipient_emails JSON,                         -- List of email addresses
  
  -- Output format
  output_format VARCHAR(20) DEFAULT 'PDF',      -- PDF, Excel, CSV
  
  -- Status
  is_active BOOLEAN DEFAULT TRUE,
  last_run_at TIMESTAMP,
  next_run_at TIMESTAMP,
  
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  
  FOREIGN KEY (report_id) REFERENCES report_definitions(report_id),
  
  INDEX idx_report (report_id),
  INDEX idx_next_run (next_run_at)
);
```

---

## 7. Analytics & Dashboards

### 7.1 Dashboard Overview

**Purpose:** Hiển thị KPIs và metrics quan trọng

**Content:**
```
┌─────────────────────────────────────────────────┐
│  DASHBOARD - 19/11/2023                       │
└─────────────────────────────────────────────────┘

┌─────────────────────┬─────────────────────┐
│ HÔM NAY            │ THÁNG NÀY           │
├─────────────────────┼─────────────────────┤
│ Bệnh nhân: 156     │ Bệnh nhân: 3,245   │
│ Doanh thu: 125.4M  │ Doanh thu: 3.2B     │
│ Lịch hẹn: 180      │ Lịch hẹn: 4,567    │
│ No-show: 15 (8.3%) │ No-show: 234 (5.1%)│
└─────────────────────┴─────────────────────┘

BIỂU ĐỒ DOANH THU (7 NGÀY GẦN ĐÂY):
     │
150M │                    ╭─╮
     │              ╭─╮   │ │
100M │        ╭─╮   │ │   │ │
     │   ╭─╮  │ │   │ │   │ │
 50M │   │ │  │ │   │ │   │ │
     └───┴─┴──┴─┴───┴─┴───┴─┴──
     13  14  15  16  17  18  19

TOP 5 CHẨN ĐOÁN:
1. Tăng huyết áp       245 BN
2. Đái tháo đường      189 BN
3. Cảm cúm            156 BN
4. Viêm phế quản       134 BN
5. Đau đầu             98 BN
```

### 7.2 Key Performance Indicators (KPIs)

**Operational KPIs:**
- Patient volume (số lượng bệnh nhân)
- Appointment utilization rate (tỷ lệ sử dụng lịch hẹn)
- No-show rate (tỷ lệ không đến)
- Average wait time (thời gian chờ trung bình)
- Doctor productivity (năng suất bác sĩ)

**Financial KPIs:**
- Daily/Monthly revenue (doanh thu)
- Revenue per patient (doanh thu/bệnh nhân)
- Collection rate (tỷ lệ thu tiền)
- Outstanding balance (công nợ)

**Clinical KPIs:**
- Treatment success rate (tỷ lệ thành công điều trị)
- Patient satisfaction (độ hài lòng)
- Readmission rate (tỷ lệ tái nhập viện)

---

## 8. Data Export

### 8.1 Export Formats

**Supported formats:**
- **PDF**: For printing, official reports
- **Excel**: For analysis, manipulation
- **CSV**: For data import, simple analysis
- **JSON**: For API integration

### 8.2 Export Scenarios

**Patient Data Export:**
```
Export: Patient List
Format: Excel
Filters:
- Date range: 01/11/2023 - 30/11/2023
- Department: Cardiology
- Status: Active

Columns:
- Patient ID
- Full Name
- Date of Birth
- Phone
- Last Visit Date
- Diagnosis
```

**Financial Data Export:**
```
Export: Financial Transactions
Format: CSV
Filters:
- Date range: 01/11/2023 - 30/11/2023
- Payment method: All

Columns:
- Transaction ID
- Date
- Patient Name
- Amount
- Payment Method
- Bill Number
```

---

## 9. Report Access Control

### 9.1 Role-Based Report Access

**Doctor:**
- Clinical reports (Patient Summary, Diagnosis, Treatment Outcomes)
- Own patient data only

**Nurse:**
- Limited clinical reports
- Assigned patients only

**Receptionist:**
- Operational reports (Daily Activity, Appointments)
- No financial data

**Cashier:**
- Financial reports (Daily Revenue, Payments)
- No clinical data

**Manager:**
- All reports
- Full access

---

## 10. Performance Considerations

### 10.1 Report Optimization

**Problem:** Reports chạy chậm với large dataset

**Solutions:**
1. **Index database** properly
2. **Cache** frequently accessed reports
3. **Pre-calculate** metrics (daily summary tables)
4. **Pagination** for large reports
5. **Background processing** for heavy reports

### 10.2 Pre-calculated Summary Tables

```sql
-- Daily summary table (pre-calculated)
CREATE TABLE daily_summaries (
  summary_date DATE PRIMARY KEY,
  
  -- Patient metrics
  new_patients INT DEFAULT 0,
  total_visits INT DEFAULT 0,
  follow_ups INT DEFAULT 0,
  
  -- Appointment metrics
  total_appointments INT DEFAULT 0,
  completed_appointments INT DEFAULT 0,
  no_shows INT DEFAULT 0,
  
  -- Financial metrics
  total_revenue DECIMAL(15,2) DEFAULT 0,
  cash_revenue DECIMAL(15,2) DEFAULT 0,
  bhytt_revenue DECIMAL(15,2) DEFAULT 0,
  
  -- Service counts
  consultations INT DEFAULT 0,
  lab_tests INT DEFAULT 0,
  prescriptions INT DEFAULT 0,
  
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  
  INDEX idx_date (summary_date)
);

-- Update daily summary (run at end of day)
INSERT INTO daily_summaries (summary_date, ...)
SELECT 
  DATE(created_at) as summary_date,
  COUNT(DISTINCT CASE WHEN is_new THEN patient_id END) as new_patients,
  COUNT(*) as total_visits,
  ...
FROM encounters
WHERE DATE(created_at) = CURDATE()
GROUP BY DATE(created_at);
```

---

## Summary

**Key takeaways:**

1. **Multiple report types**: Clinical, Operational, Financial, Administrative
2. **Real-time vs Scheduled**: Tùy nhu cầu
3. **Role-based access**: Đảm bảo security
4. **Export formats**: PDF, Excel, CSV, JSON
5. **Performance**: Pre-calculate, cache, optimize queries
6. **Dashboards**: Visual KPIs cho quick insights
7. **Audit trail**: Track report access

**For implementation:**
- Start với basic reports (Daily Activity, Daily Revenue)
- Add scheduled reports
- Implement dashboard với KPIs
- Export functionality
- Performance optimization

**Next**: [09-vietnam-compliance-regulations.md](09-vietnam-compliance-regulations.md)

