# Diagnostics & Lab Tests Workflow
# Quy trình xét nghiệm và chẩn đoán

## Overview (Tổng quan)

Lab tests và diagnostic imaging là công cụ quan trọng giúp bác sĩ:
- Xác định chẩn đoán
- Monitor bệnh mãn tính
- Screen cho các bệnh

**Common tests:**
- **Lab tests**: CBC, BMP, LFT, Lipid panel, HbA1c, Urinalysis...
- **Imaging**: X-ray, Ultrasound, CT, MRI...
- **Other**: ECG, Spirometry...

---

## 1. Lab Workflow

### 1.1 Complete Flow

```
BƯỚC 1: Doctor Orders Lab Tests
        (Bác sĩ ra chỉ định xét nghiệm)
        - Trong consultation screen
        - Select tests cần làm
        - Ghi lý do (indication)
              ↓
BƯỚC 2: Order Sent to Lab
        (Chỉ định gửi sang Lab)
        - Tự động hoặc print request form
        - Bệnh nhân nhận phiếu xét nghiệm
              ↓
BƯỚC 3: Patient Goes to Lab
        (Bệnh nhân đến phòng xét nghiệm)
        - Check-in tại lab
        - Lab tech confirm order
              ↓
BƯỚC 4: Specimen Collection
        (Lấy mẫu)
        - Lấy máu, nước tiểu, etc.
        - Label specimens
        - Ghi nhận trong system
              ↓
BƯỚC 5: Run Tests
        (Chạy xét nghiệm)
        - Lab machines process specimens
        - Results auto-import (nếu có integration)
        - Hoặc lab tech nhập manual
              ↓
BƯỚC 6: Result Review & Validation
        (Kiểm tra kết quả)
        - Lab supervisor review
        - Quality control check
        - Approve results
              ↓
BƯỚC 7: Results Sent to EMR
        (Kết quả gửi về EMR)
        - Tự động appear trong patient chart
        - Doctor receives notification
              ↓
BƯỚC 8: Doctor Reviews Results
        (Bác sĩ xem kết quả)
        - Interpret results
        - Add to assessment/plan
        - Communicate với bệnh nhân
              ↓
BƯỚC 9: Results Available to Patient
        (Bệnh nhân xem kết quả)
        - Via patient portal/app
        - Or print at front desk
```

---

## 2. Data Model

### 2.1 Lab Orders

```sql
CREATE TABLE lab_orders (
  order_id INT PRIMARY KEY AUTO_INCREMENT,
  encounter_id INT NOT NULL,
  patient_id INT NOT NULL,
  ordered_by INT NOT NULL,                       -- Doctor ID

  -- Order details
  order_number VARCHAR(20) UNIQUE NOT NULL,      -- Số phiếu XN
  order_datetime DATETIME NOT NULL,
  order_status ENUM('Pending', 'Collected', 'In Process', 'Completed', 'Cancelled')
               DEFAULT 'Pending',

  -- Clinical info
  indication TEXT,                               -- Lý do xét nghiệm
  urgent BOOLEAN DEFAULT FALSE,                  -- Stat/urgent order

  -- Timestamps
  collected_datetime DATETIME,                   -- Giờ lấy mẫu
  resulted_datetime DATETIME,                    -- Giờ có kết quả

  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,

  FOREIGN KEY (encounter_id) REFERENCES encounters(encounter_id),
  FOREIGN KEY (patient_id) REFERENCES patients(patient_id),
  FOREIGN KEY (ordered_by) REFERENCES users(user_id),

  INDEX idx_patient (patient_id),
  INDEX idx_encounter (encounter_id),
  INDEX idx_status (order_status),
  INDEX idx_order_date (order_datetime)
);
```

### 2.2 Lab Order Items

```sql
CREATE TABLE lab_order_items (
  item_id INT PRIMARY KEY AUTO_INCREMENT,
  order_id INT NOT NULL,

  -- Test info
  test_code VARCHAR(20) NOT NULL,                -- Mã xét nghiệm
  test_name VARCHAR(255) NOT NULL,               -- Tên xét nghiệm
  test_category VARCHAR(50),                     -- Hóa sinh, Huyết học, Vi sinh...

  -- Specimen
  specimen_type VARCHAR(50),                     -- Blood, Urine, Stool...
  specimen_collected BOOLEAN DEFAULT FALSE,

  -- Pricing
  price DECIMAL(10,2),

  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,

  FOREIGN KEY (order_id) REFERENCES lab_orders(order_id) ON DELETE CASCADE,

  INDEX idx_order (order_id),
  INDEX idx_test_code (test_code)
);
```

### 2.3 Lab Results

```sql
CREATE TABLE lab_results (
  result_id INT PRIMARY KEY AUTO_INCREMENT,
  order_id INT NOT NULL,
  order_item_id INT NOT NULL,
  patient_id INT NOT NULL,

  -- Test identification
  test_code VARCHAR(20) NOT NULL,
  test_name VARCHAR(255) NOT NULL,

  -- Result
  result_value VARCHAR(255),                     -- Giá trị kết quả
  result_unit VARCHAR(50),                       -- Đơn vị (mg/dL, mmol/L...)
  reference_range VARCHAR(100),                  -- Normal range
  abnormal_flag ENUM('Normal', 'High', 'Low', 'Critical High', 'Critical Low'),

  -- Result details
  result_datetime DATETIME NOT NULL,
  result_status ENUM('Preliminary', 'Final', 'Corrected', 'Cancelled')
                DEFAULT 'Preliminary',

  -- Lab info
  performed_by INT,                              -- Lab tech ID
  approved_by INT,                               -- Supervisor ID
  approved_datetime DATETIME,

  -- Notes
  result_notes TEXT,                             -- Comments from lab

  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,

  FOREIGN KEY (order_id) REFERENCES lab_orders(order_id),
  FOREIGN KEY (order_item_id) REFERENCES lab_order_items(item_id),
  FOREIGN KEY (patient_id) REFERENCES patients(patient_id),
  FOREIGN KEY (performed_by) REFERENCES users(user_id),
  FOREIGN KEY (approved_by) REFERENCES users(user_id),

  INDEX idx_order (order_id),
  INDEX idx_patient (patient_id),
  INDEX idx_test_code (test_code),
  INDEX idx_result_date (result_datetime)
);
```

### 2.4 Lab Test Catalog

```sql
CREATE TABLE lab_test_catalog (
  test_code VARCHAR(20) PRIMARY KEY,
  test_name_en VARCHAR(255),
  test_name_vi VARCHAR(255) NOT NULL,
  test_category VARCHAR(50),                     -- Hóa sinh, Huyết học...

  -- Specimen requirements
  specimen_type VARCHAR(50),                     -- Blood, Urine...
  specimen_volume VARCHAR(50),                   -- 5ml, 10ml...
  container_type VARCHAR(50),                    -- Red top, Purple top...

  -- Reference ranges (can be age/gender specific)
  reference_range_male VARCHAR(100),
  reference_range_female VARCHAR(100),
  reference_range_pediatric VARCHAR(100),

  -- Turnaround time
  tat_hours INT,                                 -- Expected TAT in hours

  -- Pricing
  base_price DECIMAL(10,2),
  bhyt_price DECIMAL(10,2),

  -- LOINC mapping (optional)
  loinc_code VARCHAR(20),

  is_active BOOLEAN DEFAULT TRUE,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,

  INDEX idx_category (test_category),
  INDEX idx_name (test_name_vi)
);
```

---

## 3. Common Lab Tests & Panels

### 3.1 Complete Blood Count (CBC - Công thức máu)

| Test | Normal Range | Ý nghĩa |
|------|--------------|---------|
| WBC (White Blood Cell) | 4.0-11.0 x10³/µL | Bạch cầu - fight infection |
| RBC (Red Blood Cell) | M: 4.5-5.5, F: 4.0-5.0 x10⁶/µL | Hồng cầu - carry oxygen |
| Hemoglobin (Hgb) | M: 13-17, F: 12-15 g/dL | Hemoglobin |
| Hematocrit (Hct) | M: 40-50%, F: 36-44% | % hồng cầu trong máu |
| Platelets | 150-400 x10³/µL | Tiểu cầu - blood clotting |

**Khi nào order CBC:**
- Sốt, nhiễm trùng
- Thiếu máu (anemia)
- Chảy máu bất thường
- Screening trước phẫu thuật

### 3.2 Basic Metabolic Panel (BMP - Chuyển hóa cơ bản)

| Test | Normal Range | Ý nghĩa |
|------|--------------|---------|
| Glucose (Fasting) | 70-100 mg/dL | Đường huyết |
| Sodium (Na) | 136-145 mEq/L | Natri |
| Potassium (K) | 3.5-5.0 mEq/L | Kali |
| Chloride (Cl) | 98-107 mEq/L | Clo |
| CO2 | 23-29 mEq/L | Bicarbonate |
| BUN | 7-20 mg/dL | Blood Urea Nitrogen - kidney function |
| Creatinine | M: 0.7-1.2, F: 0.6-1.1 mg/dL | Creatinine - kidney function |

**Khi nào order BMP:**
- Check kidney function
- Electrolyte imbalance
- Diabetes monitoring
- Dehydration

### 3.3 Liver Function Test (LFT - Chức năng gan)

| Test | Normal Range | Ý nghĩa |
|------|--------------|---------|
| ALT (SGPT) | 7-56 U/L | Enzyme gan |
| AST (SGOT) | 10-40 U/L | Enzyme gan |
| ALP | 44-147 U/L | Alkaline Phosphatase |
| Total Bilirubin | 0.1-1.2 mg/dL | Bilirubin toàn phần |
| Albumin | 3.5-5.5 g/dL | Protein |

**Khi nào order LFT:**
- Nghi ngờ bệnh gan
- Monitoring thuốc có tính gan độc
- Vàng da

### 3.4 Lipid Panel (Mỡ máu)

| Test | Optimal Range | Ý nghĩa |
|------|---------------|---------|
| Total Cholesterol | < 200 mg/dL | Cholesterol tổng |
| LDL (Bad cholesterol) | < 100 mg/dL | "Cholesterol xấu" |
| HDL (Good cholesterol) | > 60 mg/dL | "Cholesterol tốt" |
| Triglycerides | < 150 mg/dL | Triglyceride |

**Khi nào order Lipid panel:**
- Screening tim mạch
- Follow-up statin therapy
- Diabetes

### 3.5 HbA1c (Đái tháo đường)

| Result | Interpretation |
|--------|----------------|
| < 5.7% | Normal |
| 5.7-6.4% | Prediabetes |
| ≥ 6.5% | Diabetes |

**Khi nào order HbA1c:**
- Screening/diagnosis diabetes
- Monitor diabetes control (every 3 months)

### 3.6 Urinalysis (Xét nghiệm nước tiểu)

**Parameters:**
- Color, Clarity
- pH, Specific Gravity
- Protein, Glucose, Ketones
- Blood, Leukocytes, Nitrites
- Microscopy: RBC, WBC, Bacteria, Casts

**Khi nào order Urinalysis:**
- UTI symptoms
- Kidney disease screening
- Diabetes monitoring

---

## 4. UI/UX Design

### 4.1 Lab Order Screen (Doctor)

```
┌─ LAB ORDER - Nguyễn Văn An ──────────────────────┐
│                                                    │
│  Quick Panels:                                    │
│  [CBC] [BMP] [LFT] [Lipid] [Urinalysis]          │
│                                                    │
│  ─────────────────────────────────────────────────│
│                                                    │
│  🔍 Search tests: [_____________]                 │
│                                                    │
│  Selected Tests:                                  │
│  ☑ Complete Blood Count (CBC)      150,000 VND   │
│  ☑ Blood Glucose (Fasting)          50,000 VND   │
│  ☑ HbA1c                            200,000 VND   │
│  ☐ Lipid Panel                      180,000 VND   │
│                                                    │
│  Total: 400,000 VND                               │
│                                                    │
│  Indication: [Đái tháo đường - follow up____]    │
│                                                    │
│  ☐ Urgent/Stat                                    │
│                                                    │
│  [Cancel]  [Order Labs]                           │
└────────────────────────────────────────────────────┘
```

### 4.2 Lab Specimen Collection Screen

```
┌─ LAB SPECIMEN COLLECTION ─────────────────────────┐
│  Patient: Nguyễn Văn An (P2023001234)            │
│  Order #: LAB20231119001                          │
└────────────────────────────────────────────────────┘

Tests Ordered:
┌────────────────────────────────────────────────────┐
│ ☑ CBC - Blood (Purple top, 3ml)     [Collected]  │
│ ☑ Glucose - Blood (Gray top, 2ml)   [Collected]  │
│ ☑ HbA1c - Blood (Purple top, 2ml)   [Collected]  │
└────────────────────────────────────────────────────┘

Collection Details:
  Collected by: Nguyễn Thị B (Lab Tech)
  Collection time: 19/11/2023 08:30
  Fasting: ☑ Yes  ☐ No

  Notes: [Patient fasted 10 hours]

  [Print Labels]  [Mark Complete]
```

### 4.3 Lab Results View (Doctor)

```
┌─ LAB RESULTS - Nguyễn Văn An ─────────────────────┐
│  Order Date: 19/11/2023 08:15                     │
│  Result Date: 19/11/2023 10:30                    │
│  Status: ✓ Final                                  │
└────────────────────────────────────────────────────┘

COMPLETE BLOOD COUNT (CBC)
┌────────────────────────────────────────────────────┐
│ Test        Result    Unit      Range      Flag   │
├────────────────────────────────────────────────────│
│ WBC         12.5      x10³/µL   4.0-11.0   🔺HIGH │
│ RBC         4.8       x10⁶/µL   4.5-5.5    Normal │
│ Hemoglobin  14.2      g/dL      13-17      Normal │
│ Hematocrit  42%       %          40-50     Normal │
│ Platelets   250       x10³/µL   150-400    Normal │
└────────────────────────────────────────────────────┘

GLUCOSE & DIABETES
┌────────────────────────────────────────────────────┐
│ Glucose (F) 180       mg/dL     70-100     🔺HIGH │
│ HbA1c       8.5%      %          <5.7      🔺HIGH │
└────────────────────────────────────────────────────┘

Interpretation: [Add doctor's interpretation______]

[Print Results]  [Add to Note]  [Order Follow-up]
```

---

## 5. Integration với Lab Machines

### 5.1 Lab Information System (LIS) Integration

**HL7 messaging cho lab orders & results:**

**Order message (ORM^O01):**
```
MSH|^~\&|EMR|Clinic|LIS|Lab|20231119083000||ORM^O01|MSG001|P|2.5
PID|1||P2023001234||NGUYEN^VAN^A||19850315|M
ORC|NW|LAB20231119001|||||||20231119083000
OBR|1|LAB20231119001||CBC^Complete Blood Count|||20231119083000
OBR|2|LAB20231119001||GLU^Glucose|||20231119083000
```

**Result message (ORU^R01):**
```
MSH|^~\&|LIS|Lab|EMR|Clinic|20231119103000||ORU^R01|MSG002|P|2.5
PID|1||P2023001234||NGUYEN^VAN^A||19850315|M
OBR|1|LAB20231119001||CBC^Complete Blood Count|||20231119083000|||F
OBX|1|NM|WBC^White Blood Cell||12.5|x10^3/uL|4.0-11.0|H|||F
OBX|2|NM|RBC^Red Blood Cell||4.8|x10^6/uL|4.5-5.5|N|||F
OBX|3|NM|HGB^Hemoglobin||14.2|g/dL|13-17|N|||F
```

### 5.2 Auto-import Results

```javascript
// Parse HL7 ORU message and import results
async function importLabResults(hl7Message) {
  const parsed = hl7Parser.parse(hl7Message);

  // Extract patient
  const patientMRN = parsed.PID.patientID;
  const patient = await findPatientByMRN(patientMRN);

  // Extract order
  const orderNumber = parsed.OBR.placerOrderNumber;
  const labOrder = await findLabOrder(orderNumber);

  // Extract results (OBX segments)
  for (const obx of parsed.OBX) {
    const result = {
      order_id: labOrder.order_id,
      patient_id: patient.patient_id,
      test_code: obx.observationIdentifier.identifier,
      test_name: obx.observationIdentifier.text,
      result_value: obx.observationValue,
      result_unit: obx.units,
      reference_range: obx.referenceRange,
      abnormal_flag: obx.abnormalFlags, // N, H, L, HH, LL
      result_datetime: obx.observationDateTime,
      result_status: 'Final'
    };

    await saveLabResult(result);
  }

  // Notify doctor
  await notifyDoctor(labOrder.ordered_by, `Lab results available for ${patient.full_name}`);
}
```

---

## 6. Critical Value Alerts

**Critical values** cần thông báo bác sĩ NGAY LẬP TỨC:

| Test | Critical Low | Critical High |
|------|--------------|---------------|
| Glucose | < 50 mg/dL | > 400 mg/dL |
| Potassium (K) | < 2.5 mEq/L | > 6.0 mEq/L |
| Sodium (Na) | < 120 mEq/L | > 160 mEq/L |
| WBC | < 2.0 x10³/µL | > 30.0 x10³/µL |
| Platelets | < 50 x10³/µL | > 1000 x10³/µL |
| Creatinine | - | > 5.0 mg/dL |

**Alert mechanism:**
```javascript
function checkCriticalValue(result) {
  const criticalRanges = getCriticalRanges(result.test_code);

  if (!criticalRanges) return null;

  const value = parseFloat(result.result_value);

  if (value < criticalRanges.low || value > criticalRanges.high) {
    return {
      isCritical: true,
      severity: 'CRITICAL',
      message: `CRITICAL: ${result.test_name} = ${result.result_value} ${result.result_unit}`,
      action: 'Notify doctor immediately'
    };
  }

  return null;
}

// When critical value detected
async function handleCriticalValue(result) {
  // 1. Send urgent notification to doctor
  await sendUrgentNotification(result.ordered_by, {
    title: 'CRITICAL LAB VALUE',
    message: `${result.patient_name}: ${result.test_name} = ${result.result_value}`,
    priority: 'high'
  });

  // 2. SMS/Call to doctor (configurable)
  await sendSMS(result.doctor_phone, `CRITICAL: ${result.patient_name} - ${result.test_name}: ${result.result_value}`);

  // 3. Log critical alert
  await logCriticalAlert(result);

  // 4. Flag in EMR with red banner
  await flagPatientChart(result.patient_id, 'CRITICAL_LAB_VALUE');
}
```

---

## 7. Imaging Orders & Results

### 7.1 Common Imaging Studies

**X-Ray (Chụp X-quang)**:
- Chest X-ray (CXR) - Phổi
- Abdominal X-ray - Bụng
- Spine X-ray - Cột sống
- Extremities - Tứ chi

**Ultrasound (Siêu âm)**:
- Abdominal ultrasound - Bụng
- Pelvic ultrasound - Tiểu khung
- Thyroid ultrasound - Tuyến giáp
- Obstetric ultrasound - Thai

**CT Scan**:
- CT Brain - Não
- CT Chest - Ngực
- CT Abdomen/Pelvis - Bụng/chậu

**MRI**:
- MRI Brain - Não
- MRI Spine - Cột sống
- MRI joints - Khớp

### 7.2 Imaging Order Data Model

```sql
CREATE TABLE imaging_orders (
  order_id INT PRIMARY KEY AUTO_INCREMENT,
  encounter_id INT NOT NULL,
  patient_id INT NOT NULL,
  ordered_by INT NOT NULL,

  -- Order details
  order_number VARCHAR(20) UNIQUE NOT NULL,
  imaging_type ENUM('X-Ray', 'Ultrasound', 'CT', 'MRI', 'Other'),
  study_name VARCHAR(255) NOT NULL,              -- "Chest X-ray", "Abdominal Ultrasound"...

  -- Clinical info
  indication TEXT,                               -- Lý do chụp
  clinical_history TEXT,

  -- Scheduling
  order_datetime DATETIME NOT NULL,
  scheduled_datetime DATETIME,
  performed_datetime DATETIME,

  -- Status
  order_status ENUM('Pending', 'Scheduled', 'In Progress', 'Completed', 'Cancelled')
               DEFAULT 'Pending',

  -- Results
  report_text TEXT,                              -- Radiologist's report
  reported_by INT,                               -- Radiologist ID
  reported_datetime DATETIME,

  -- PACS integration
  study_instance_uid VARCHAR(255),               -- DICOM Study UID
  images_available BOOLEAN DEFAULT FALSE,

  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,

  FOREIGN KEY (encounter_id) REFERENCES encounters(encounter_id),
  FOREIGN KEY (patient_id) REFERENCES patients(patient_id),
  FOREIGN KEY (ordered_by) REFERENCES users(user_id),
  FOREIGN KEY (reported_by) REFERENCES users(user_id),

  INDEX idx_patient (patient_id),
  INDEX idx_status (order_status)
);
```

---

## 8. Quality Control

### 8.1 Lab QC Checks

**Before releasing results:**
1. **Delta check**: Compare với previous results
   - Nếu thay đổi quá lớn → alert lab tech
2. **Reference range validation**
3. **Critical value alert**
4. **Supervisor approval** cho critical/abnormal results

### 8.2 Result Amendment

**Nếu phát hiện sai:**
```sql
-- Original result
UPDATE lab_results
SET result_status = 'Corrected',
    corrected_datetime = NOW()
WHERE result_id = 123;

-- Insert corrected result
INSERT INTO lab_results (
  order_id, test_code, result_value, result_status, ...
) VALUES (
  456, 'GLU', 120, 'Final', ...
);

-- Log amendment
INSERT INTO result_amendments (
  original_result_id, corrected_result_id,
  reason, amended_by
) VALUES (123, 789, 'Sample mix-up', user_id);
```

---

## Summary

**Key takeaways:**

1. **Lab workflow**: Order → Collect → Process → Result → Review
2. **Common panels**: CBC, BMP, LFT, Lipid, HbA1c, Urinalysis
3. **Critical values**: Must alert doctor immediately
4. **Integration**: HL7 messaging với LIS, auto-import results
5. **QC**: Delta checks, supervisor approval, result amendments
6. **UI**: Easy ordering (panels), clear result display với flags

**For implementation:**
- Start with test catalog (common tests)
- Manual result entry first, then add HL7 integration
- Critical value alerts are MUST-HAVE (patient safety)
- Result trending (show previous results for comparison)
- Mobile-friendly result viewer

**Next**: [04-prescription-pharmacy.md](04-prescription-pharmacy.md)
