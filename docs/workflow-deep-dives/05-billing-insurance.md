# Billing & Insurance Workflow
# Quy trình thanh toán và bảo hiểm y tế

## Overview (Tổng quan)

Billing & Insurance là workflow quan trọng cho revenue của phòng khám/bệnh viện. Sai sót trong billing có thể dẫn đến:
- Mất revenue (undercharging)
- Mất uy tín (overcharging)
- Reject claims từ BHYT → không được thanh toán
- Vi phạm quy định → phạt

**Mục tiêu**: Chính xác, minh bạch, tuân thủ quy định

---

## 1. Billing Workflow

### 1.1 Complete Flow

```
BƯỚC 1: Service Delivery
        (Cung cấp dịch vụ)
        - Bác sĩ khám bệnh
        - Xét nghiệm, chụp chiếu
        - Kê đơn thuốc
        - Thủ thuật...
              ↓
BƯỚC 2: Charge Capture
        (Ghi nhận chi phí)
        - Tự động từ orders (lab, meds, procedures)
        - Manual entry (consultation fee)
        - Apply pricing rules
              ↓
BƯỚC 3: Bill Generation
        (Tạo hóa đơn)
        - Aggregate all charges
        - Calculate totals
        - Apply discounts/packages (if any)
              ↓
BƯỚC 4: Insurance Verification & Adjudication
        (Kiểm tra bảo hiểm)
        - Check BHYT coverage
        - Calculate patient responsibility
        - Calculate insurance responsibility
              ↓
BƯỚC 5: Bill Presentation
        (Trình bày hóa đơn cho bệnh nhân)
        - Itemized bill
        - Insurance portion
        - Patient portion
              ↓
BƯỚC 6: Payment Collection
        (Thu tiền)
        - Cash, Card, Bank transfer
        - Record payment
        - Issue receipt
              ↓
BƯỚC 7: Insurance Claim Submission
        (Nộp hồ sơ bảo hiểm)
        - Generate BHYT XML
        - Submit to BHXH
        - Track claim status
              ↓
BƯỚC 8: Claim Adjudication (by Insurance)
        (Bảo hiểm xét duyệt)
        - Approve, Deny, or Partial approval
              ↓
BƯỚC 9: Payment Posting
        (Ghi nhận thanh toán từ bảo hiểm)
        - Record insurance payment
        - Handle denials/rejections
              ↓
BƯỚC 10: Reconciliation
         (Đối soát)
         - Match payments with bills
         - Identify outstanding balances
         - Follow up on unpaid claims
```

---

## 2. Data Model

### 2.1 Bills/Invoices

```sql
CREATE TABLE bills (
  bill_id INT PRIMARY KEY AUTO_INCREMENT,
  encounter_id INT NOT NULL,
  patient_id INT NOT NULL,

  -- Bill info
  bill_number VARCHAR(20) UNIQUE NOT NULL,       -- Số hóa đơn
  bill_date DATE NOT NULL,
  bill_type ENUM('Outpatient', 'Inpatient', 'Emergency') DEFAULT 'Outpatient',

  -- Amounts
  subtotal DECIMAL(12,2) NOT NULL,               -- Tổng trước giảm giá
  discount_amount DECIMAL(12,2) DEFAULT 0,       -- Giảm giá
  total_amount DECIMAL(12,2) NOT NULL,           -- Tổng sau giảm giá

  -- Insurance
  insurance_amount DECIMAL(12,2) DEFAULT 0,      -- Phần BHYT chi trả
  patient_amount DECIMAL(12,2) NOT NULL,         -- Phần bệnh nhân chi trả

  -- Payment status
  payment_status ENUM('Unpaid', 'Partially Paid', 'Paid', 'Cancelled')
                 DEFAULT 'Unpaid',
  amount_paid DECIMAL(12,2) DEFAULT 0,
  amount_due DECIMAL(12,2),                      -- Số tiền còn nợ

  -- Insurance claim
  insurance_claimed BOOLEAN DEFAULT FALSE,
  claim_number VARCHAR(20),                      -- Số hồ sơ BHYT
  claim_status ENUM('Pending', 'Submitted', 'Approved', 'Partially Approved', 'Denied'),
  claim_submitted_date DATE,
  claim_paid_date DATE,

  -- Notes
  notes TEXT,

  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  created_by INT,

  FOREIGN KEY (encounter_id) REFERENCES encounters(encounter_id),
  FOREIGN KEY (patient_id) REFERENCES patients(patient_id),
  FOREIGN KEY (created_by) REFERENCES users(user_id),

  INDEX idx_patient (patient_id),
  INDEX idx_encounter (encounter_id),
  INDEX idx_bill_date (bill_date),
  INDEX idx_payment_status (payment_status),
  INDEX idx_claim_status (claim_status)
);
```

### 2.2 Bill Items (Chi tiết hóa đơn)

```sql
CREATE TABLE bill_items (
  item_id INT PRIMARY KEY AUTO_INCREMENT,
  bill_id INT NOT NULL,

  -- Service/Product
  item_type ENUM('Consultation', 'Lab Test', 'Imaging', 'Medication', 'Procedure', 'Other'),
  item_code VARCHAR(20),                         -- Mã dịch vụ/thuốc
  item_name VARCHAR(255) NOT NULL,
  item_category VARCHAR(100),

  -- Quantity & Price
  quantity DECIMAL(10,2) DEFAULT 1,
  unit_price DECIMAL(10,2) NOT NULL,
  total_price DECIMAL(10,2) NOT NULL,            -- quantity * unit_price

  -- Insurance coverage
  bhyt_covered BOOLEAN DEFAULT FALSE,            -- Có thuộc BHYT không
  bhyt_price DECIMAL(10,2),                      -- Giá BHYT
  bhyt_percentage DECIMAL(5,2),                  -- % BHYT chi trả (80%, 95%, 100%)
  bhyt_amount DECIMAL(10,2),                     -- Số tiền BHYT chi trả
  patient_copay DECIMAL(10,2),                   -- Số tiền bệnh nhân tự trả

  -- Reference
  reference_type VARCHAR(50),                    -- 'Prescription', 'Lab Order', 'Encounter'
  reference_id INT,                              -- ID của prescription/lab order...

  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,

  FOREIGN KEY (bill_id) REFERENCES bills(bill_id) ON DELETE CASCADE,

  INDEX idx_bill (bill_id),
  INDEX idx_item_type (item_type),
  INDEX idx_item_code (item_code)
);
```

### 2.3 Payments

```sql
CREATE TABLE payments (
  payment_id INT PRIMARY KEY AUTO_INCREMENT,
  bill_id INT NOT NULL,
  patient_id INT NOT NULL,

  -- Payment info
  payment_number VARCHAR(20) UNIQUE NOT NULL,    -- Số biên lai
  payment_date DATE NOT NULL,
  payment_amount DECIMAL(12,2) NOT NULL,

  -- Payment method
  payment_method ENUM('Cash', 'Card', 'Bank Transfer', 'Insurance', 'Other'),
  card_type VARCHAR(50),                         -- Visa, Mastercard...
  transaction_reference VARCHAR(100),            -- Transaction ID

  -- Source
  payment_source ENUM('Patient', 'Insurance') DEFAULT 'Patient',

  -- Received by
  received_by INT,                               -- Cashier user ID

  notes TEXT,

  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,

  FOREIGN KEY (bill_id) REFERENCES bills(bill_id),
  FOREIGN KEY (patient_id) REFERENCES patients(patient_id),
  FOREIGN KEY (received_by) REFERENCES users(user_id),

  INDEX idx_bill (bill_id),
  INDEX idx_patient (patient_id),
  INDEX idx_payment_date (payment_date),
  INDEX idx_payment_method (payment_method)
);
```

### 2.4 Service Price List

```sql
CREATE TABLE service_price_list (
  service_code VARCHAR(20) PRIMARY KEY,
  service_name_vi VARCHAR(255) NOT NULL,
  service_name_en VARCHAR(255),
  service_category VARCHAR(100),                 -- Khám bệnh, Xét nghiệm, Chẩn đoán hình ảnh...

  -- Pricing
  base_price DECIMAL(10,2) NOT NULL,             -- Giá niêm yết
  vat_applicable BOOLEAN DEFAULT FALSE,          -- Có tính VAT không
  vat_rate DECIMAL(5,2) DEFAULT 10.00,           -- % VAT

  -- BHYT
  bhyt_covered BOOLEAN DEFAULT FALSE,
  bhyt_code VARCHAR(20),                         -- Mã dịch vụ theo BHYT
  bhyt_price DECIMAL(10,2),                      -- Giá BHYT quy định

  -- Validity
  effective_date DATE,
  expiry_date DATE,
  is_active BOOLEAN DEFAULT TRUE,

  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,

  INDEX idx_category (service_category),
  INDEX idx_name (service_name_vi)
);
```

---

## 3. BHYT (Vietnam Social Insurance) Specifics

### 3.1 BHYT Coverage Levels

**Quyền lợi BHYT:**

| Mã đầu thẻ | Đối tượng | % Chi trả | Miễn cùng chi trả |
|------------|-----------|-----------|-------------------|
| GD | Gia đình | 80% | Không |
| HC | Học sinh, sinh viên | 100% | Có |
| TE | Trẻ em dưới 6 tuổi | 100% | Có |
| TS | Người có công | 100% | Có |
| KC | Người nghèo, cận nghèo | 100% | Có |
| CB | Cán bộ, công chức | 95% | Không |
| XH | Bảo trợ xã hội | 100% | Có |
| CN | Người lao động | 80% | Không |

**Cùng chi trả:**
- 20% của giá dịch vụ (cho mã GD, CN, CB...)
- Maximum co-pay per visit: 6 months minimum wage

### 3.2 BHYT Card Validation

```javascript
// Validate BHYT card number (15 digits)
function validateBHYTCard(cardNumber) {
  // Format: AB1234567890123
  // AB: Mã quyền lợi (2 ký tự)
  // 1: Mã tỉnh (1 số)
  // 23: Năm cấp (2 số)
  // 4567890123: Số thứ tự (10 số)

  if (cardNumber.length !== 15) return { valid: false, error: 'Invalid length' };

  const regex = /^[A-Z]{2}\d{13}$/;
  if (!regex.test(cardNumber)) return { valid: false, error: 'Invalid format' };

  const prefix = cardNumber.substring(0, 2);
  const validPrefixes = ['GD', 'HC', 'TE', 'TS', 'KC', 'CB', 'XH', 'CN', 'QN', 'CA'];

  if (!validPrefixes.includes(prefix)) {
    return { valid: false, error: 'Invalid prefix code' };
  }

  return {
    valid: true,
    prefix: prefix,
    coverageLevel: getCoverageLevel(prefix)
  };
}

function getCoverageLevel(prefix) {
  const levels = {
    'GD': { percentage: 80, coPaymentExemption: false },
    'HC': { percentage: 100, coPaymentExemption: true },
    'TE': { percentage: 100, coPaymentExemption: true },
    'TS': { percentage: 100, coPaymentExemption: true },
    'KC': { percentage: 100, coPaymentExemption: true },
    'CB': { percentage: 95, coPaymentExemption: false },
    'XH': { percentage: 100, coPaymentExemption: true },
    'CN': { percentage: 80, coPaymentExemption: false }
  };

  return levels[prefix] || { percentage: 80, coPaymentExemption: false };
}
```

### 3.3 BHYT Claim XML Generation

**BHXH yêu cầu XML format cụ thể:**

**XML1 - Thông tin hành chính:**
```xml
<THONGTINHANHCHINH>
  <MACSKCB>12345</MACSKCB>                    <!-- Mã cơ sở KCB -->
  <SOTHE>GD1234567890123</SOTHE>              <!-- Số thẻ BHYT -->
  <HOTEN>NGUYEN VAN A</HOTEN>
  <NGAYSINH>15/03/1985</NGAYSINH>
  <GIOITINH>1</GIOITINH>                      <!-- 1: Nam, 2: Nữ -->
  <DIACHI>123 Le Loi, Q1, TPHCM</DIACHI>
  <MATHE>GD123</MATHE>                        <!-- Mã quyền lợi -->
  <GTTHETU>01/01/2023</GTTHETU>               <!-- Giá trị thẻ từ -->
  <GTTHEDEN>31/12/2023</GTTHEDEN>             <!-- Giá trị thẻ đến -->
  <NGAYVAO>19/11/2023 08:30</NGAYVAO>         <!-- Ngày vào viện/khám -->
  <NGAYRA>19/11/2023 10:00</NGAYRA>           <!-- Ngày ra viện/khám xong -->
  <CHANDOAN>K29.0</CHANDOAN>                  <!-- ICD-10 chẩn đoán -->
  <MAICD10>K29.0</MAICD10>
</THONGTINHANHCHINH>
```

**XML2 - Chi tiết chi phí:**
```xml
<TONGHOP>
  <T_TONGCHI>500000</T_TONGCHI>              <!-- Tổng chi phí -->
  <T_BHTT>400000</T_BHTT>                    <!-- BHYT thanh toán -->
  <T_BNTT>100000</T_BNTT>                    <!-- Bệnh nhân thanh toán -->
</TONGHOP>
```

**XML3 - Chi tiết thuốc:**
```xml
<THUOC>
  <MATHUOC>MED001</MATHUOC>
  <TENTHUOC>PARACETAMOL 500MG</TENTHUOC>
  <DONVI>VIEN</DONVI>
  <SOLUONG>21</SOLUONG>
  <DONGIA>5000</DONGIA>
  <THANHTIEN>105000</THANHTIEN>
  <MUCHUONG>80</MUCHUONG>                    <!-- % BHYT chi trả -->
  <TYLE>80</TYLE>
  <TTBHYT>84000</TTBHYT>                     <!-- Số tiền BHYT trả -->
  <BNCT>21000</BNCT>                         <!-- Bệnh nhân cùng chi trả -->
</THUOC>
```

**Generate XML function:**
```javascript
async function generateBHYTClaimXML(encounterId) {
  const encounter = await getEncounterDetails(encounterId);
  const patient = encounter.patient;
  const bill = encounter.bill;
  const billItems = bill.items;

  // XML1 - Administrative info
  const xml1 = `
    <THONGTINHANHCHINH>
      <MACSKCB>${clinic.code}</MACSKCB>
      <SOTHE>${patient.insurance_card_number}</SOTHE>
      <HOTEN>${patient.full_name.toUpperCase()}</HOTEN>
      <NGAYSINH>${formatDate(patient.dob)}</NGAYSINH>
      <GIOITINH>${patient.gender === 'M' ? 1 : 2}</GIOITINH>
      <DIACHI>${patient.address}</DIACHI>
      <GTTHETU>${formatDate(patient.insurance_valid_from)}</GTTHETU>
      <GTTHEDEN>${formatDate(patient.insurance_valid_to)}</GTTHEDEN>
      <NGAYVAO>${formatDateTime(encounter.arrival_datetime)}</NGAYVAO>
      <NGAYRA>${formatDateTime(encounter.checkout_datetime)}</NGAYRA>
      <CHANDOAN>${encounter.primary_diagnosis_icd10}</CHANDOAN>
    </THONGTINHANHCHINH>
  `;

  // XML2 - Totals
  const xml2 = `
    <TONGHOP>
      <T_TONGCHI>${bill.total_amount}</T_TONGCHI>
      <T_BHTT>${bill.insurance_amount}</T_BHTT>
      <T_BNTT>${bill.patient_amount}</T_BNTT>
    </TONGHOP>
  `;

  // XML3 - Medications
  const medications = billItems.filter(item => item.item_type === 'Medication');
  const xml3 = medications.map(med => `
    <THUOC>
      <MATHUOC>${med.item_code}</MATHUOC>
      <TENTHUOC>${med.item_name}</TENTHUOC>
      <SOLUONG>${med.quantity}</SOLUONG>
      <DONGIA>${med.unit_price}</DONGIA>
      <THANHTIEN>${med.total_price}</THANHTIEN>
      <TTBHYT>${med.bhyt_amount}</TTBHYT>
      <BNCT>${med.patient_copay}</BNCT>
    </THUOC>
  `).join('');

  // XML4 - Services
  // ... similar to XML3 for services

  // Combine all XMLs
  const fullXML = `<?xml version="1.0" encoding="UTF-8"?>
    <HOSOKHAMCHUABENH>
      ${xml1}
      ${xml2}
      ${xml3}
      <!-- XML4, XML5... -->
    </HOSOKHAMCHUABENH>
  `;

  return fullXML;
}
```

---

## 4. Billing Calculations

### 4.1 Calculate Bill with BHYT

```javascript
async function calculateBillWithBHYT(encounterId) {
  const encounter = await getEncounterDetails(encounterId);
  const patient = encounter.patient;
  const billItems = encounter.billItems;

  let subtotal = 0;
  let bhytTotal = 0;
  let patientTotal = 0;

  for (const item of billItems) {
    const itemTotal = item.quantity * item.unit_price;
    subtotal += itemTotal;

    if (item.bhyt_covered && patient.has_insurance) {
      // Check if service is covered by BHYT
      const bhytPrice = item.bhyt_price || item.unit_price;
      const coverageLevel = patient.insurance_coverage_level; // 80%, 95%, 100%

      let bhytAmount = bhytPrice * item.quantity * (coverageLevel / 100);

      // Co-payment (cùng chi trả) nếu không miễn
      let patientCopay = 0;
      if (!patient.copayment_exemption) {
        patientCopay = bhytPrice * item.quantity * ((100 - coverageLevel) / 100);
      }

      // Patient pays the difference if price > BHYT price
      const priceDifference = (item.unit_price - bhytPrice) * item.quantity;
      const patientAmount = patientCopay + Math.max(0, priceDifference);

      item.bhyt_amount = bhytAmount;
      item.patient_copay = patientAmount;

      bhytTotal += bhytAmount;
      patientTotal += patientAmount;

    } else {
      // Not covered by BHYT → patient pays full
      item.bhyt_amount = 0;
      item.patient_copay = itemTotal;
      patientTotal += itemTotal;
    }
  }

  return {
    subtotal: subtotal,
    insurance_amount: bhytTotal,
    patient_amount: patientTotal,
    total_amount: subtotal
  };
}
```

**Example:**
```
Consultation fee: 200,000 VND (BHYT price: 150,000, coverage: 80%)
- BHYT pays: 150,000 * 80% = 120,000 VND
- Patient copay (20%): 150,000 * 20% = 30,000 VND
- Price difference: 200,000 - 150,000 = 50,000 VND
- Patient total: 30,000 + 50,000 = 80,000 VND

Paracetamol: 105,000 VND (BHYT price: 100,000, coverage: 80%)
- BHYT pays: 100,000 * 80% = 80,000 VND
- Patient copay: 100,000 * 20% = 20,000 VND
- Price difference: 5,000 VND
- Patient total: 20,000 + 5,000 = 25,000 VND

Total bill: 305,000 VND
BHYT pays: 200,000 VND
Patient pays: 105,000 VND
```

---

## 5. UI/UX Design

### 5.1 Bill Summary Screen

```
┌─ HÓA ĐƠN #INV20231119001 ──────────────────────────┐
│  Bệnh nhân: Nguyễn Văn An (P2023001234)           │
│  Ngày: 19/11/2023                                  │
│  BHYT: GD1234567890123 (80% - Hết hạn: 31/12/2023)│
└─────────────────────────────────────────────────────┘

CHI TIẾT DỊCH VỤ:
┌─────────────────────────────────────────────────────┐
│ Dịch vụ            SL  Đơn giá    Thành tiền  BHYT │
├─────────────────────────────────────────────────────│
│ Khám bệnh          1   200,000    200,000   120,000│
│ Công thức máu      1   150,000    150,000   120,000│
│ Đường huyết        1    50,000     50,000    40,000│
└─────────────────────────────────────────────────────┘

THUỐC:
┌─────────────────────────────────────────────────────┐
│ Paracetamol 500mg  21   5,000     105,000    84,000│
│ Omeprazole 20mg    28  10,000     280,000   224,000│
└─────────────────────────────────────────────────────┘

TỔNG KẾT:
┌─────────────────────────────────────────────────────┐
│ Tổng cộng:                              785,000 VND │
│ BHYT chi trả (80%):                     588,000 VND │
│ ───────────────────────────────────────────────────│
│ Bệnh nhân thanh toán:                   197,000 VND │
└─────────────────────────────────────────────────────┘

[In hóa đơn]  [Thu tiền]  [Hủy]
```

### 5.2 Payment Screen

```
┌─ THU TIỀN - Hóa đơn #INV20231119001 ───────────────┐
│  Bệnh nhân: Nguyễn Văn An                          │
│  Số tiền phải trả: 197,000 VND                     │
└─────────────────────────────────────────────────────┘

Phương thức thanh toán:
○ Tiền mặt (Cash)
○ Thẻ (Card)
○ Chuyển khoản (Bank Transfer)

Số tiền nhận: [197,000______________] VND
Tiền thừa:    [0______________________] VND

Ghi chú: [___________________________________]

[Hủy]  [Xác nhận thanh toán]
```

### 5.3 Receipt

```
┌─────────────────────────────────────────────────────┐
│            PHÒNG KHÁM ĐA KHOA ABC                   │
│         123 Lê Lợi, Quận 1, TPHCM                   │
│           Tel: 028-1234-5678                        │
│              MST: 0123456789                        │
├─────────────────────────────────────────────────────┤
│              BIÊN LAI THU TIỀN                      │
│           Receipt #: REC20231119001                 │
├─────────────────────────────────────────────────────┤
│ Ngày: 19/11/2023        Giờ: 10:30                 │
│                                                     │
│ Họ tên: NGUYỄN VĂN AN                              │
│ Mã BN: P2023001234                                 │
│ Địa chỉ: 123 Nguyễn Huệ, Q1, TPHCM                 │
│                                                     │
│ Lý do thu: Viện phí khám bệnh                      │
│                                                     │
│ Tổng hóa đơn:                      785,000 VND     │
│ BHYT chi trả:                      588,000 VND     │
│ ─────────────────────────────────────────────────  │
│ Bệnh nhân thanh toán:              197,000 VND     │
│                                                     │
│ Bằng chữ: Một trăm chín mươi bảy nghìn đồng       │
│                                                     │
│ Hình thức: Tiền mặt                                │
├─────────────────────────────────────────────────────┤
│ Thu ngân: Nguyễn Thị B                             │
│ Chữ ký:                                            │
│                                                     │
│         Cảm ơn quý khách!                          │
└─────────────────────────────────────────────────────┘
```

---

## 6. Revenue Cycle Management

### 6.1 Key Metrics

**Metrics to track:**
- **Collection Rate**: (Amount collected / Amount billed) * 100%
- **Days in AR**: Average days to collect payment
- **Denial Rate**: (Denied claims / Total claims) * 100%
- **Net Collection Rate**: (Amount collected / (Amount billed - Contractual adjustments)) * 100%

```javascript
// Calculate collection rate
function calculateCollectionRate(startDate, endDate) {
  const billed = getTotalBilled(startDate, endDate);
  const collected = getTotalCollected(startDate, endDate);
  return (collected / billed) * 100;
}

// Calculate average days in AR
function calculateDaysInAR() {
  const totalAR = getTotalAccountsReceivable();
  const avgDailyRevenue = getAverageDailyRevenue(last90Days);
  return totalAR / avgDailyRevenue;
}
```

### 6.2 Outstanding Bills Report

```sql
SELECT
  bill_number,
  patient_name,
  bill_date,
  total_amount,
  amount_paid,
  amount_due,
  DATEDIFF(CURDATE(), bill_date) as days_outstanding
FROM bills
JOIN patients USING (patient_id)
WHERE payment_status IN ('Unpaid', 'Partially Paid')
  AND amount_due > 0
ORDER BY days_outstanding DESC;
```

**Aging report:**
| Age Bucket | Count | Total Amount |
|------------|-------|--------------|
| 0-30 days | 50 | 25,000,000 VND |
| 31-60 days | 20 | 15,000,000 VND |
| 61-90 days | 10 | 8,000,000 VND |
| 90+ days | 5 | 5,000,000 VND |

### 6.3 Claim Denial Management

**Common denial reasons:**
1. **Incomplete information**: Missing ICD-10 code, missing service date...
2. **Not covered**: Service not in BHYT catalog
3. **Expired insurance**: BHYT card expired
4. **Wrong facility**: Patient registered at different facility
5. **Exceeded limits**: Over annual/per-visit limit

**Denial workflow:**
```
Claim denied by BHXH
    ↓
Review denial reason
    ↓
Is it correctable?
    ↓
  YES → Fix and resubmit
    ↓
  NO → Write off or bill patient
```

---

## 7. Discounts & Packages

### 7.1 Discount Types

```sql
CREATE TABLE discounts (
  discount_id INT PRIMARY KEY AUTO_INCREMENT,
  discount_code VARCHAR(20) UNIQUE,
  discount_name VARCHAR(255),

  -- Type
  discount_type ENUM('Percentage', 'Fixed Amount'),
  discount_value DECIMAL(10,2),                  -- 10% or 50,000 VND

  -- Applicability
  applies_to ENUM('Entire Bill', 'Specific Services', 'Specific Categories'),
  applicable_services TEXT,                      -- JSON array of service codes

  -- Validity
  valid_from DATE,
  valid_to DATE,
  is_active BOOLEAN DEFAULT TRUE,

  -- Usage limits
  max_uses INT,
  current_uses INT DEFAULT 0,

  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

**Example discounts:**
- Senior citizen discount: 10% off
- Loyalty member discount: 15% off
- Seasonal promotion: 100,000 VND off for packages > 1,000,000 VND

### 7.2 Package Pricing

**Health checkup packages:**

```sql
CREATE TABLE service_packages (
  package_id INT PRIMARY KEY AUTO_INCREMENT,
  package_code VARCHAR(20) UNIQUE,
  package_name VARCHAR(255),
  package_description TEXT,

  -- Included services (JSON array)
  included_services TEXT,                        -- [{"code": "XN001", "name": "CBC"}, ...]

  -- Pricing
  regular_price DECIMAL(10,2),                   -- Sum of individual prices
  package_price DECIMAL(10,2),                   -- Discounted package price
  savings DECIMAL(10,2),                         -- regular_price - package_price

  valid_from DATE,
  valid_to DATE,
  is_active BOOLEAN DEFAULT TRUE
);
```

**Example package:**
```
Gói khám sức khỏe tổng quát (Basic Health Checkup)
- Khám nội khoa
- Khám mắt
- Công thức máu (CBC)
- Đường huyết
- Mỡ máu
- X-quang phổi
- Siêu âm bụng

Giá lẻ: 1,500,000 VND
Giá gói: 1,200,000 VND
Tiết kiệm: 300,000 VND (20%)
```

---

## 8. Reports

### 8.1 Daily Revenue Report

```sql
SELECT
  DATE(bill_date) as date,
  COUNT(*) as total_bills,
  SUM(total_amount) as total_revenue,
  SUM(insurance_amount) as insurance_revenue,
  SUM(patient_amount) as cash_revenue,
  SUM(amount_paid) as collected,
  SUM(amount_due) as outstanding
FROM bills
WHERE bill_date BETWEEN '2023-11-01' AND '2023-11-30'
  AND payment_status != 'Cancelled'
GROUP BY DATE(bill_date)
ORDER BY date;
```

### 8.2 Revenue by Service Category

```sql
SELECT
  item_category,
  COUNT(*) as total_services,
  SUM(total_price) as revenue
FROM bill_items
JOIN bills USING (bill_id)
WHERE bill_date BETWEEN '2023-11-01' AND '2023-11-30'
GROUP BY item_category
ORDER BY revenue DESC;
```

### 8.3 Top Revenue Services

```sql
SELECT
  item_code,
  item_name,
  COUNT(*) as times_ordered,
  SUM(quantity) as total_quantity,
  SUM(total_price) as total_revenue
FROM bill_items
JOIN bills USING (bill_id)
WHERE bill_date BETWEEN '2023-11-01' AND '2023-11-30'
GROUP BY item_code, item_name
ORDER BY total_revenue DESC
LIMIT 10;
```

---

## 9. Payment Gateway Integration (for Card/Online)

### 9.1 Payment Flow

```
Patient selects Card payment
    ↓
Redirect to payment gateway
    ↓
Patient enters card details
    ↓
Gateway processes payment
    ↓
Callback to EMR with result
    ↓
Record payment in EMR
    ↓
Display receipt
```

### 9.2 Integration Example

```javascript
// Initiate payment
async function initiateCardPayment(billId, amount) {
  const bill = await getBill(billId);

  // Call payment gateway API
  const paymentRequest = {
    merchant_id: config.merchantId,
    order_id: bill.bill_number,
    amount: amount,
    currency: 'VND',
    return_url: `${config.baseUrl}/payments/callback`,
    cancel_url: `${config.baseUrl}/payments/cancel`,
    description: `Payment for bill ${bill.bill_number}`
  };

  const response = await paymentGateway.createPayment(paymentRequest);

  // Redirect user to payment URL
  return response.payment_url;
}

// Handle callback
async function handlePaymentCallback(req) {
  const { order_id, transaction_id, status, amount } = req.body;

  // Verify signature (security)
  if (!verifySignature(req.body)) {
    throw new Error('Invalid signature');
  }

  if (status === 'SUCCESS') {
    // Record payment
    await recordPayment({
      bill_number: order_id,
      payment_amount: amount,
      payment_method: 'Card',
      transaction_reference: transaction_id,
      payment_source: 'Patient'
    });

    // Update bill status
    await updateBillStatus(order_id, 'Paid');

    return { success: true, message: 'Payment successful' };
  } else {
    return { success: false, message: 'Payment failed' };
  }
}
```

---

## Summary

**Key takeaways:**

1. **Accurate charge capture**: Tất cả dịch vụ phải được ghi nhận đầy đủ
2. **BHYT compliance**: Đúng format XML, đầy đủ thông tin
3. **Transparent billing**: Itemized bill, rõ ràng BHYT vs patient portion
4. **Multiple payment methods**: Cash, card, bank transfer
5. **Revenue cycle management**: Track outstanding, follow up denials
6. **Reports**: Daily revenue, service revenue, aging reports

**For implementation:**
- Start with service price list
- BHYT calculation logic (critical for VN)
- Generate correct BHYT XML format
- Payment collection & receipts
- Reports for management
- Integration với payment gateway (cards)

**Critical for VN clinics:**
- BHYT XML generation & submission
- Correct coverage calculation (80%, 95%, 100%)
- Co-payment calculation
- Claim tracking & denial management

---

**END OF DOCUMENTATION**

Congratulations! Bạn đã có một bộ tài liệu toàn diện về Healthcare Domain cho việc phát triển EMR/HIS cho phòng khám!
