# Patient Registration & Admission
# Đăng ký bệnh nhân & Nhập viện

## Overview (Tổng quan)

Patient Registration là **bước đầu tiên** và **quan trọng nhất** trong healthcare workflow. Thông tin đăng ký chính xác là nền tảng cho toàn bộ quá trình điều trị sau này.

**Tại sao quan trọng?**
- Sai thông tin bệnh nhân → nhầm hồ sơ, nhầm thuốc → nguy hiểm tính mạng
- Thiếu thông tin bảo hiểm → không thanh toán được BHYT
- Trùng hồ sơ (duplicate records) → dữ liệu rối, khó quản lý

---

## 1. Types of Registration (Các loại đăng ký)

### 1.1 New Patient Registration (Đăng ký bệnh nhân mới)

**Khi nào:** Lần đầu tiên bệnh nhân đến phòng khám/bệnh viện

**Thông tin cần thu thập:**

#### Thông tin cá nhân (Demographics)
- **Họ và tên** (Full name)
  - Họ (Family name): Nguyễn
  - Tên đệm (Middle name): Văn
  - Tên (Given name): An
- **Ngày sinh** (Date of birth): DD/MM/YYYY
- **Giới tính** (Gender): Nam/Nữ/Khác
- **CMND/CCCD** (National ID): 001234567890
- **Địa chỉ** (Address):
  - Địa chỉ thường trú (Permanent address)
  - Địa chỉ liên hệ (Contact address) - nếu khác
- **Số điện thoại** (Phone number): 0909123456
- **Email**: (optional nhưng nên có cho patient portal)
- **Nghề nghiệp** (Occupation)
- **Dân tộc** (Ethnicity) - required cho BHYT ở VN

#### Thông tin bảo hiểm y tế (Insurance Information)
- **Có BHYT không?** (Has insurance?)
- **Số thẻ BHYT** (Insurance card number): GD123456789012345
- **Mã CSYT** (Medical facility code nơi đăng ký KCB ban đầu)
- **Ngày bắt đầu hiệu lực** (Valid from date)
- **Ngày hết hạn** (Valid to date)
- **Quyền lợi** (Coverage level): 100%, 95%, 80%...
- **Miễn cùng chi trả** (Co-payment exemption): Có/Không

#### Người liên hệ khẩn cấp (Emergency Contact)
- Tên người liên hệ
- Quan hệ (Relationship): Vợ/Chồng, Con, Bố/Mẹ...
- Số điện thoại

#### Thông tin y tế cơ bản (Basic Medical Info)
- **Nhóm máu** (Blood type): A+, B+, O+, AB+...
- **Dị ứng thuốc** (Drug allergies): Penicillin, Aspirin...
- **Tiền sử bệnh** (Medical history):
  - Bệnh mãn tính (Chronic diseases): Tiểu đường, cao huyết áp...
  - Phẫu thuật trước đây (Previous surgeries)

**⚠️ Important validation:**
- CMND/CCCD: Check format, check duplicate
- Số thẻ BHYT: 15 digits, có algorithm validation
- Ngày sinh: Logical (không thể > ngày hiện tại)
- Số điện thoại: Vietnamese phone format (09xx, 08xx, 07xx...)

### 1.2 Returning Patient Check-in (Bệnh nhân cũ tái khám)

**Khi nào:** Bệnh nhân đã có hồ sơ, quay lại khám

**Process (Quy trình):**

```
Bước 1: Tìm bệnh nhân
        (Patient search/lookup)
        - Tìm theo: Tên, SĐT, CMND, Mã bệnh nhân
              ↓
Bước 2: Xác nhận thông tin
        (Verify information)
        - Kiểm tra: Tên, ngày sinh, địa chỉ còn đúng không?
        - Cập nhật nếu có thay đổi
              ↓
Bước 3: Kiểm tra BHYT (nếu có)
        (Verify insurance)
        - Còn hiệu lực không?
        - Đúng nơi đăng ký KCB không?
              ↓
Bước 4: Chọn lý do khám
        (Select visit reason)
        - Khám bệnh
        - Tái khám
        - Khám định kỳ
              ↓
Bước 5: Tạo encounter mới
        (Create new encounter/visit)
        - Generate visit number
        - Assign to queue
```

**⚠️ Common issues:**
- **Duplicate patients**: Cùng 1 người nhưng 2 hồ sơ (typo tên, số điện thoại khác...)
  - Solution: Patient matching algorithm, merge function
- **Outdated insurance**: BHYT hết hạn
  - Solution: Real-time validation, alert front desk
- **Wrong facility code**: Bệnh nhân đăng ký BHYT ở bệnh viện khác
  - Solution: Check mã CSYT, hướng dẫn chuyển tuyến nếu cần

### 1.3 Appointment-based Registration (Đăng ký theo lịch hẹn)

**Khi nào:** Bệnh nhân đã đặt lịch trước (online hoặc phone)

**Process:**
```
Bước 1: Bệnh nhân đến theo lịch hẹn
              ↓
Bước 2: Front desk tra cứu appointment
        - Theo: Tên, SĐT, Appointment ID
              ↓
Bước 3: Check-in appointment
        - Mark as "arrived"
        - Update actual arrival time
              ↓
Bước 4: Confirm/Update thông tin
              ↓
Bước 5: Proceed to triage/consultation
```

**⚠️ Edge cases:**
- Bệnh nhân đến sớm/muộn
- Bệnh nhân không đến (no-show)
- Bác sĩ trễ/hủy lịch

---

## 2. Data Model (Mô hình dữ liệu)

### 2.1 Patient Table

```sql
CREATE TABLE patients (
  -- Primary Key
  patient_id INT PRIMARY KEY AUTO_INCREMENT,
  medical_record_number VARCHAR(20) UNIQUE NOT NULL,  -- Mã bệnh án (MRN)

  -- Demographics
  family_name VARCHAR(50) NOT NULL,           -- Họ
  middle_name VARCHAR(50),                    -- Tên đệm
  given_name VARCHAR(50) NOT NULL,            -- Tên
  full_name VARCHAR(150) NOT NULL,            -- Họ và tên đầy đủ
  full_name_no_diacritics VARCHAR(150),       -- Không dấu (for search)

  gender ENUM('M', 'F', 'O') NOT NULL,        -- Nam/Nữ/Khác
  date_of_birth DATE NOT NULL,

  -- Identifiers
  national_id VARCHAR(12) UNIQUE,             -- CMND/CCCD
  passport_number VARCHAR(20),                -- Hộ chiếu (foreigners)

  -- Contact
  phone_number VARCHAR(15),
  email VARCHAR(100),

  -- Address
  address_line1 VARCHAR(255),                 -- Số nhà, đường
  address_line2 VARCHAR(255),                 -- Phường/Xã
  district VARCHAR(100),                      -- Quận/Huyện
  city VARCHAR(100),                          -- Tỉnh/Thành phố
  postal_code VARCHAR(10),
  country VARCHAR(2) DEFAULT 'VN',

  -- Other demographics
  occupation VARCHAR(100),                    -- Nghề nghiệp
  ethnicity VARCHAR(50),                      -- Dân tộc

  -- Medical info
  blood_type VARCHAR(5),                      -- A+, B+, O+, AB+, A-, B-, O-, AB-

  -- Emergency contact
  emergency_contact_name VARCHAR(100),
  emergency_contact_relationship VARCHAR(50),
  emergency_contact_phone VARCHAR(15),

  -- System fields
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  created_by INT,                             -- User ID
  is_active BOOLEAN DEFAULT TRUE,             -- Soft delete

  -- Indexes
  INDEX idx_full_name (full_name),
  INDEX idx_full_name_no_diacritics (full_name_no_diacritics),
  INDEX idx_phone (phone_number),
  INDEX idx_dob (date_of_birth),
  INDEX idx_national_id (national_id)
);
```

### 2.2 Insurance Table

```sql
CREATE TABLE patient_insurance (
  insurance_id INT PRIMARY KEY AUTO_INCREMENT,
  patient_id INT NOT NULL,

  -- Insurance info
  insurance_type ENUM('BHYT', 'Private', 'None') DEFAULT 'None',
  insurance_card_number VARCHAR(15),          -- Số thẻ BHYT (15 digits)

  -- BHYT specific
  registered_facility_code VARCHAR(10),       -- Mã CSYT đăng ký KCB ban đầu
  coverage_level DECIMAL(5,2),                -- 100.00, 95.00, 80.00...
  co_payment_exemption BOOLEAN DEFAULT FALSE, -- Miễn cùng chi trả

  -- Validity
  valid_from DATE,
  valid_to DATE,
  is_active BOOLEAN DEFAULT TRUE,

  -- Timestamps
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,

  FOREIGN KEY (patient_id) REFERENCES patients(patient_id),
  INDEX idx_patient (patient_id),
  INDEX idx_card_number (insurance_card_number)
);
```

### 2.3 Allergies & Medical History

```sql
CREATE TABLE patient_allergies (
  allergy_id INT PRIMARY KEY AUTO_INCREMENT,
  patient_id INT NOT NULL,

  allergen_type ENUM('Drug', 'Food', 'Environmental', 'Other'),
  allergen_name VARCHAR(255) NOT NULL,        -- Tên thuốc/thực phẩm gây dị ứng
  reaction VARCHAR(500),                      -- Phản ứng: nổi mẩn, khó thở...
  severity ENUM('Mild', 'Moderate', 'Severe'),

  onset_date DATE,                            -- Ngày phát hiện
  notes TEXT,

  is_active BOOLEAN DEFAULT TRUE,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,

  FOREIGN KEY (patient_id) REFERENCES patients(patient_id),
  INDEX idx_patient (patient_id)
);

CREATE TABLE patient_medical_history (
  history_id INT PRIMARY KEY AUTO_INCREMENT,
  patient_id INT NOT NULL,

  condition_type ENUM('Chronic Disease', 'Past Surgery', 'Past Illness', 'Family History'),
  icd10_code VARCHAR(10),                     -- Mã ICD-10
  condition_name VARCHAR(255) NOT NULL,
  diagnosis_date DATE,
  notes TEXT,

  is_active BOOLEAN DEFAULT TRUE,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,

  FOREIGN KEY (patient_id) REFERENCES patients(patient_id),
  INDEX idx_patient (patient_id),
  INDEX idx_icd10 (icd10_code)
);
```

---

## 3. User Interface Design (Thiết kế giao diện)

### 3.1 Registration Form Layout

**Nguyên tắc thiết kế:**
- ✅ Form ngắn gọn, rõ ràng (minimize fields)
- ✅ Group related fields together
- ✅ Required fields được đánh dấu rõ (*)
- ✅ Inline validation (validate ngay khi nhập)
- ✅ Autocomplete cho địa chỉ
- ✅ Date picker cho ngày sinh

**Form sections:**

```
┌─────────────────────────────────────────────────┐
│  ĐĂNG KÝ BỆNH NHÂN MỚI                          │
└─────────────────────────────────────────────────┘

┌─ Thông tin cá nhân ─────────────────────────────┐
│                                                  │
│  Họ *: [___________]  Tên đệm: [___________]   │
│  Tên *: [___________]                           │
│                                                  │
│  Ngày sinh *: [DD/MM/YYYY]  Giới tính *: [▼]   │
│  CMND/CCCD: [____________]                      │
│                                                  │
│  Số điện thoại *: [___________]                 │
│  Email: [___________]                            │
└──────────────────────────────────────────────────┘

┌─ Địa chỉ ────────────────────────────────────────┐
│                                                  │
│  Địa chỉ: [_________________________________]   │
│  Phường/Xã: [______________]                    │
│  Quận/Huyện: [▼]  Tỉnh/TP: [▼]                 │
└──────────────────────────────────────────────────┘

┌─ Bảo hiểm y tế ──────────────────────────────────┐
│                                                  │
│  ☐ Có BHYT                                      │
│  Số thẻ: [_______________] [Validate]          │
│  Hiệu lực: [DD/MM/YYYY] đến [DD/MM/YYYY]       │
│  Mã CSYT: [__________]                          │
└──────────────────────────────────────────────────┘

┌─ Người liên hệ khẩn cấp ─────────────────────────┐
│                                                  │
│  Tên: [___________]  Quan hệ: [▼]              │
│  SĐT: [___________]                             │
└──────────────────────────────────────────────────┘

    [Hủy]  [Lưu và tiếp tục]
```

### 3.2 Patient Search Interface

**Tìm kiếm nhanh:**
```
┌─────────────────────────────────────────────────┐
│  Tìm bệnh nhân:                                 │
│  [🔍 Tên, SĐT, CMND, Mã BN...]                 │
└─────────────────────────────────────────────────┘

Kết quả:
┌─────────────────────────────────────────────────┐
│  Nguyễn Văn An - Nam - 15/03/1985              │
│  MÃ BN: P000123  | SĐT: 0909123456             │
│  ───────────────────────────────────────────────│
│  Nguyễn Văn Anh - Nữ - 20/05/1990             │
│  MÃ BN: P000456  | SĐT: 0912345678             │
└─────────────────────────────────────────────────┘
```

**Search algorithm:**
1. Exact match trước (national ID, MRN)
2. Fuzzy match sau (name với soundex/metaphone)
3. Search cả có dấu và không dấu
4. Highlight matching text

---

## 4. Business Logic & Validation

### 4.1 Medical Record Number (MRN) Generation

**Format suggestions:**
- **Option 1**: P + Year + Sequential (P2023001234)
- **Option 2**: Clinic code + Year + Sequential (CL12023001234)
- **Option 3**: UUID/GUID (for distributed systems)

**Implementation:**
```javascript
// Example: P + YYYY + 6-digit sequential
function generateMRN() {
  const year = new Date().getFullYear();
  const sequence = getNextSequence('mrn', year); // From DB
  const mrn = `P${year}${sequence.toString().padStart(6, '0')}`;
  return mrn;
}
// Result: P2023000001, P2023000002...
```

**⚠️ Important:**
- MRN phải unique across toàn hệ thống
- Không được tái sử dụng MRN cũ (kể cả khi delete bệnh nhân)
- Transaction-safe (avoid race conditions)

### 4.2 BHYT Card Number Validation

**Format BHYT ở VN:**
- 15 ký tự: `GD1234567890123`
- 2 ký tự đầu: Mã quyền lợi (GD, HC, TE, ...)
- 1 ký tự: Mã tỉnh
- 2 ký tự: Năm cấp
- 10 ký tự cuối: Số thứ tự

**Validation algorithm:**
```javascript
function validateBHYT(cardNumber) {
  // Check length
  if (cardNumber.length !== 15) return false;

  // Check format (letters + digits)
  const regex = /^[A-Z]{2}\d{1}\d{2}\d{10}$/;
  if (!regex.test(cardNumber)) return false;

  // Check mã quyền lợi (2 ký tự đầu)
  const validPrefixes = ['GD', 'HC', 'TE', 'TS', 'KC', 'CB', 'XH', 'CN', 'QN', 'CA'];
  const prefix = cardNumber.substring(0, 2);
  if (!validPrefixes.includes(prefix)) return false;

  // More validation logic...
  return true;
}
```

**Real-time validation:**
- Gọi API BHXH để check thẻ còn hiệu lực (nếu có)
- VN hiện chưa có public API, phải dùng manual validation

### 4.3 Duplicate Patient Detection

**Matching criteria:**
- **Exact match**: National ID, Phone number
- **Fuzzy match**: Name + DOB (soundex, Levenshtein distance)

**Algorithm:**
```javascript
function findPotentialDuplicates(patient) {
  const duplicates = [];

  // 1. Exact match on National ID
  if (patient.nationalId) {
    const exactMatch = db.query(
      'SELECT * FROM patients WHERE national_id = ?',
      [patient.nationalId]
    );
    if (exactMatch) duplicates.push({
      patient: exactMatch,
      confidence: 'HIGH',
      reason: 'Same National ID'
    });
  }

  // 2. Name + DOB match
  const nameMatches = db.query(`
    SELECT *, SOUNDEX(full_name) as name_soundex
    FROM patients
    WHERE date_of_birth = ?
    AND SOUNDEX(full_name) = SOUNDEX(?)
  `, [patient.dob, patient.fullName]);

  nameMatches.forEach(match => {
    duplicates.push({
      patient: match,
      confidence: 'MEDIUM',
      reason: 'Same name and DOB'
    });
  });

  // 3. Phone number match
  if (patient.phone) {
    const phoneMatch = db.query(
      'SELECT * FROM patients WHERE phone_number = ?',
      [patient.phone]
    );
    if (phoneMatch) duplicates.push({
      patient: phoneMatch,
      confidence: 'MEDIUM',
      reason: 'Same phone number'
    });
  }

  return duplicates;
}
```

**UI flow:**
```
User nhập thông tin → Before save → Run duplicate detection
                                            ↓
                                    Found potential duplicates?
                                            ↓
                                          YES
                                            ↓
                            Show warning modal:
                            "Có thể trùng với bệnh nhân sau:
                             - Nguyễn Văn An (P2023001234)
                               DOB: 15/03/1985, SĐT: 0909123456

                             [Sử dụng hồ sơ này] [Vẫn tạo mới]"
```

---

## 5. Integration Points (Điểm tích hợp)

### 5.1 BHYT Validation Service

**Ideal flow (nếu có API):**
```
Front desk nhập số thẻ BHYT
          ↓
Call BHXH API để validate
          ↓
Response:
- Valid/Invalid
- Coverage level
- Valid dates
- Registered facility
          ↓
Auto-fill thông tin vào form
```

**Current reality (VN):**
- Chưa có public API từ BHXH
- Validation thủ công hoặc dùng offline database
- Some hospitals có kết nối riêng với BHXH

### 5.2 National ID Verification

**Potential integration:**
- Quét CCCD chip (NFC reader)
- OCR để đọc thông tin từ ảnh CMND/CCCD
- Auto-fill name, DOB, address

**Benefits:**
- Giảm lỗi nhập liệu
- Nhanh hơn manual entry

---

## 6. Workflow Variations (Biến thể quy trình)

### 6.1 Walk-in vs Appointment

**Walk-in (Không hẹn trước):**
```
Patient arrives → Registration → Queue → Wait → See doctor
```
- First-come, first-served
- Có thể chờ lâu

**Appointment (Đã đặt lịch):**
```
Patient arrives → Quick check-in → Priority queue → See doctor
```
- Thời gian cố định
- Ít chờ đợi hơn

### 6.2 Emergency vs Non-Emergency

**Emergency (Cấp cứu):**
- **Skip** nhiều bước registration
- Collect minimum info (name, age, emergency contact)
- Complete registration sau khi stabilize

**Non-emergency (Khám thường):**
- Full registration process

### 6.3 Self-Service Kiosk

**Xu hướng mới:**
- Bệnh nhân tự đăng ký qua kiosk/tablet
- Scan thẻ BHYT, CCCD
- Confirm thông tin
- Print queue number

**Benefits:**
- Giảm tải cho front desk
- Nhanh hơn
- Patient privacy (không cần nói to thông tin)

---

## 7. Common Issues & Solutions

### Issue 1: Thông tin không đầy đủ
**Problem:** Bệnh nhân không nhớ/không có CMND, không biết địa chỉ chính xác...
**Solution:**
- Required fields tối thiểu: Name, DOB, Phone
- Cho phép update sau
- Mark incomplete records với flag để follow-up

### Issue 2: Bệnh nhân quên mang thẻ BHYT
**Problem:** Có BHYT nhưng quên mang thẻ
**Solution:**
- Cho phép tạm đăng ký không BHYT
- Có thể bổ sung sau trong X ngày
- Hướng dẫn cách check BHYT online (VssID app)

### Issue 3: Trùng tên, trùng ngày sinh
**Problem:** 2 người khác nhau nhưng tên và DOB giống nhau
**Solution:**
- Thêm identifier khác: National ID, phone
- Photo của bệnh nhân trong hồ sơ
- Confirm với bệnh nhân trước khi proceed

### Issue 4: Thông tin thay đổi (địa chỉ, SĐT...)
**Problem:** Bệnh nhân chuyển nhà, đổi SĐT
**Solution:**
- Always ask "Thông tin còn đúng không?" khi check-in
- Easy update flow
- Version history (track changes)

---

## 8. Performance Considerations

### Patient Search Optimization

**Problem:** Database có 100,000+ patients → search chậm

**Solutions:**
1. **Full-text search indexing**
```sql
ALTER TABLE patients ADD FULLTEXT INDEX idx_ft_name (full_name, full_name_no_diacritics);
```

2. **Elasticsearch/Solr** cho search phức tạp
3. **Caching** cho frequently accessed patients
4. **Pagination** (không load hết results)

### Concurrent Registration

**Problem:** Peak hours, nhiều front desk cùng register patients → conflicts

**Solutions:**
- Database transactions với proper isolation level
- Optimistic locking cho updates
- Queue system để manage load

---

## Summary

**Key takeaways:**

1. **Registration là foundation** - sai ở đây → sai cả workflow sau
2. **Validate thoroughly**: BHYT card, National ID, duplicates
3. **Minimize data entry**: Autocomplete, defaults, integrations
4. **Search phải nhanh và accurate**: Fuzzy matching, multiple criteria
5. **Handle edge cases**: Emergency, incomplete info, duplicates
6. **Vietnam-specific**: BHYT validation, dân tộc, mã CSYT

**For implementation:**
- Start simple (name, DOB, phone, gender)
- Add complexity incrementally (insurance, allergies, history...)
- Test duplicate detection thoroughly
- Optimize search performance early

**Next:** Outpatient Consultation Workflow
