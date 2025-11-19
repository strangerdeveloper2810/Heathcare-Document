# Prescription & Pharmacy Workflow
# Quy trình kê đơn thuốc và nhà thuốc

## Overview (Tổng quan)

Prescription (kê đơn thuốc) là một phần quan trọng và nhạy cảm của healthcare workflow. Sai sót trong kê đơn có thể gây hậu quả nghiêm trọng:
- Sai thuốc → bệnh nhân uống nhầm
- Sai liều → quá liều hoặc không đủ liều
- Không check tương tác thuốc → phản ứng phụ nguy hiểm
- Không check dị ứng → shock phản vệ

**Mục tiêu**: An toàn, chính xác, hiệu quả

---

## 1. Prescription Workflow

### 1.1 Complete Flow

```
BƯỚC 1: Doctor Selects Medication
        (Bác sĩ chọn thuốc)
        - Search drug database
        - Select medication
              ↓
BƯỚC 2: Clinical Decision Support
        (Kiểm tra an toàn)
        - Check allergies ⚠️
        - Check drug interactions ⚠️
        - Check contraindications
        - Check duplicate therapy
              ↓
        Có alerts? → Review và quyết định
              ↓
BƯỚC 3: Specify Details
        (Ghi rõ chi tiết)
        - Dosage (liều lượng)
        - Route (đường dùng): PO, IV, IM...
        - Frequency (tần suất): QD, BID, TID...
        - Duration (thời gian): 7 days, 30 days...
        - Quantity (số lượng)
        - Instructions (hướng dẫn)
              ↓
BƯỚC 4: Review & Sign
        (Xem lại và ký)
        - Doctor reviews prescription
        - Electronic signature
              ↓
BƯỚC 5: Prescription Sent
        (Gửi đơn thuốc)
        - To internal pharmacy: Electronically
        - To external pharmacy: Print or e-prescribe
        - To patient: Print copy
              ↓
BƯỚC 6: Pharmacist Review
        (Dược sĩ kiểm tra)
        - Verify prescription
        - Check appropriateness
        - Final safety check
              ↓
BƯỚC 7: Dispense Medication
        (Cấp thuốc)
        - Locate medication in inventory
        - Count/measure correct amount
        - Label properly
        - Update inventory
              ↓
BƯỚC 8: Patient Counseling
        (Tư vấn bệnh nhân)
        - How to take medication
        - Side effects to watch for
        - Drug interactions
        - Storage instructions
              ↓
BƯỚC 9: Medication Administration Record
        (Ghi nhận)
        - Mark as dispensed
        - Record in MAR (for inpatients)
        - Patient receives medication
```

---

## 2. Data Model

### 2.1 Prescriptions Table

```sql
CREATE TABLE prescriptions (
  prescription_id INT PRIMARY KEY AUTO_INCREMENT,
  encounter_id INT NOT NULL,
  patient_id INT NOT NULL,
  prescribed_by INT NOT NULL,                    -- Doctor ID

  -- Prescription info
  prescription_number VARCHAR(20) UNIQUE NOT NULL,
  prescription_date DATE NOT NULL,

  -- Status
  status ENUM('Active', 'Completed', 'Cancelled', 'On Hold')
         DEFAULT 'Active',

  -- Valid dates
  valid_from DATE NOT NULL,
  valid_until DATE,                              -- Expiry date (e.g., 30 days from issue)

  -- Notes
  general_instructions TEXT,                     -- Overall instructions

  -- Signatures
  signed BOOLEAN DEFAULT FALSE,
  signed_at TIMESTAMP,

  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,

  FOREIGN KEY (encounter_id) REFERENCES encounters(encounter_id),
  FOREIGN KEY (patient_id) REFERENCES patients(patient_id),
  FOREIGN KEY (prescribed_by) REFERENCES users(user_id),

  INDEX idx_patient (patient_id),
  INDEX idx_encounter (encounter_id),
  INDEX idx_prescriber (prescribed_by),
  INDEX idx_status (status)
);
```

### 2.2 Prescription Items (Medication List)

```sql
CREATE TABLE prescription_items (
  item_id INT PRIMARY KEY AUTO_INCREMENT,
  prescription_id INT NOT NULL,

  -- Medication
  medication_code VARCHAR(20) NOT NULL,          -- Mã thuốc quốc gia
  medication_name VARCHAR(255) NOT NULL,
  generic_name VARCHAR(255),                     -- Tên hoạt chất
  brand_name VARCHAR(255),                       -- Tên biệt dược

  -- Dosage
  strength VARCHAR(50),                          -- 500mg, 10mg/ml...
  dosage_form VARCHAR(50),                       -- Tablet, Capsule, Syrup, Injection...

  -- Instructions (SIG)
  route VARCHAR(20),                             -- PO, IV, IM, SC, TOP, INH, PR, SL
  dose VARCHAR(50),                              -- 1 tablet, 5ml, 2 puffs...
  frequency VARCHAR(20),                         -- QD, BID, TID, QID, PRN...
  duration VARCHAR(50),                          -- 7 days, 2 weeks, 1 month...
  timing VARCHAR(100),                           -- Before meals, After meals, Bedtime...

  -- Quantity
  quantity DECIMAL(10,2),                        -- Số lượng
  unit VARCHAR(20),                              -- viên, lọ, tuýp...
  refills INT DEFAULT 0,                         -- Số lần tái cấp

  -- Instructions
  instructions TEXT,                             -- Patient instructions (Vietnamese)
  pharmacist_notes TEXT,                         -- Notes for pharmacist

  -- Pricing
  unit_price DECIMAL(10,2),
  total_price DECIMAL(10,2),

  -- Dispensing
  dispensed BOOLEAN DEFAULT FALSE,
  dispensed_quantity DECIMAL(10,2),
  dispensed_by INT,                              -- Pharmacist ID
  dispensed_at TIMESTAMP,

  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,

  FOREIGN KEY (prescription_id) REFERENCES prescriptions(prescription_id) ON DELETE CASCADE,
  FOREIGN KEY (dispensed_by) REFERENCES users(user_id),

  INDEX idx_prescription (prescription_id),
  INDEX idx_medication (medication_code)
);
```

### 2.3 Medication Catalog

```sql
CREATE TABLE medications (
  medication_code VARCHAR(20) PRIMARY KEY,       -- Mã thuốc quốc gia
  generic_name VARCHAR(255) NOT NULL,            -- Tên hoạt chất
  brand_names TEXT,                              -- Danh sách tên biệt dược (JSON array)

  -- Classification
  therapeutic_class VARCHAR(100),                -- Nhóm điều trị
  pharmacological_class VARCHAR(100),            -- Nhóm dược lý

  -- Forms available
  dosage_forms TEXT,                             -- JSON array: ["Tablet", "Capsule", "Syrup"]
  strengths TEXT,                                -- JSON array: ["250mg", "500mg"]

  -- Indications
  indications TEXT,                              -- Chỉ định
  contraindications TEXT,                        -- Chống chỉ định

  -- Dosing
  adult_dose VARCHAR(255),
  pediatric_dose VARCHAR(255),
  max_daily_dose VARCHAR(100),

  -- Safety
  pregnancy_category VARCHAR(5),                 -- A, B, C, D, X
  black_box_warning BOOLEAN DEFAULT FALSE,
  controlled_substance VARCHAR(10),              -- C-II, C-III... (US), Prescription Only (VN)

  -- Interactions (can be detailed table)
  drug_interactions TEXT,                        -- Major interactions

  -- Pricing
  base_price DECIMAL(10,2),
  bhyt_covered BOOLEAN DEFAULT FALSE,
  bhyt_percentage DECIMAL(5,2),

  -- Stock management (if internal pharmacy)
  requires_prescription BOOLEAN DEFAULT TRUE,
  requires_refrigeration BOOLEAN DEFAULT FALSE,

  -- Reference
  rxnorm_code VARCHAR(20),                       -- International code

  is_active BOOLEAN DEFAULT TRUE,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,

  INDEX idx_generic_name (generic_name),
  INDEX idx_therapeutic_class (therapeutic_class),
  FULLTEXT INDEX idx_ft_search (generic_name, brand_names)
);
```

---

## 3. Prescription Notation (SIG Codes)

### 3.1 Route (Đường dùng)

| Code | Full Term | Tiếng Việt |
|------|-----------|------------|
| PO | Per os (by mouth) | Uống |
| IV | Intravenous | Tiêm tĩnh mạch |
| IM | Intramuscular | Tiêm bắp |
| SC/SubQ | Subcutaneous | Tiêm dưới da |
| TOP | Topical | Bôi ngoài da |
| INH | Inhalation | Hít |
| PR | Per rectum | Đặt hậu môn |
| SL | Sublingual | Ngậm dưới lưỡi |
| OU/OD/OS | Both eyes/Right eye/Left eye | Nhỏ mắt |

### 3.2 Frequency (Tần suất)

| Code | Latin | Meaning | Tiếng Việt |
|------|-------|---------|------------|
| QD | Quaque die | Once daily | Ngày 1 lần |
| BID | Bis in die | Twice daily | Ngày 2 lần |
| TID | Ter in die | Three times daily | Ngày 3 lần |
| QID | Quater in die | Four times daily | Ngày 4 lần |
| Q4H | Quaque 4 hora | Every 4 hours | 4 giờ 1 lần |
| Q6H | Quaque 6 hora | Every 6 hours | 6 giờ 1 lần |
| QHS | Quaque hora somni | Every night at bedtime | Trước khi ngủ |
| PRN | Pro re nata | As needed | Khi cần |
| AC | Ante cibum | Before meals | Trước ăn |
| PC | Post cibum | After meals | Sau ăn |
| STAT | Statim | Immediately | Ngay lập tức |

### 3.3 Duration

- 7 days
- 2 weeks
- 1 month
- Until finished
- Ongoing (cho bệnh mãn tính)

---

## 4. UI/UX Design

### 4.1 Prescription Entry Screen

```
┌─ KÊ ĐƠN THUỐC - Nguyễn Văn An ────────────────────┐
│                                                     │
│  🔍 Search medication: [Paracetamol__________]     │
│                                                     │
│  Quick suggestions:                                │
│  • Paracetamol 500mg tablet                       │
│  • Paracetamol 250mg/5ml syrup                    │
│  • Paracetamol 325mg tablet                       │
│                                                     │
└─────────────────────────────────────────────────────┘

┌─ MEDICATION DETAILS ────────────────────────────────┐
│                                                     │
│  Drug: Paracetamol 500mg Tablet                    │
│                                                     │
│  Strength: [500mg▼]  Form: [Tablet▼]              │
│                                                     │
│  Route: [PO - Uống▼]                               │
│                                                     │
│  Dose: [1] [tablet▼]                               │
│                                                     │
│  Frequency: [TID (3 times/day)▼]                   │
│                                                     │
│  Timing: [☑] After meals  ☐ Before meals          │
│                                                     │
│  Duration: [7] [days▼]                             │
│                                                     │
│  Quantity: [21] tablets (auto-calculated)          │
│                                                     │
│  Instructions (Vietnamese):                         │
│  [Uống 1 viên sau ăn, ngày 3 lần, uống trong 7 ngày]│
│                                                     │
│  ⚠️  No known allergies                            │
│  ⚠️  No drug interactions found                    │
│                                                     │
│  [Cancel]  [Add to Prescription]                   │
└─────────────────────────────────────────────────────┘
```

### 4.2 Prescription Summary

```
┌─ ĐƠN THUỐC #RX20231119001 ─────────────────────────┐
│  Bệnh nhân: Nguyễn Văn An - 38 tuổi - Nam         │
│  Chẩn đoán: Viêm dạ dày cấp (K29.0)               │
│  Ngày kê: 19/11/2023                               │
│  Bác sĩ: Dr. Trần Văn C                            │
└─────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────┐
│ 1. Paracetamol 500mg Tablet                        │
│    • 1 viên x 3 lần/ngày x 7 ngày                  │
│    • Sau ăn                                         │
│    • Tổng: 21 viên                                  │
│    • Giá: 105,000 VND                              │
│                                                     │
│ 2. Omeprazole 20mg Capsule                         │
│    • 1 viên x 2 lần/ngày x 14 ngày                 │
│    • Trước ăn 30 phút                               │
│    • Tổng: 28 viên                                  │
│    • Giá: 280,000 VND                              │
│                                                     │
│ 3. Domperidone 10mg Tablet                         │
│    • 1 viên x 3 lần/ngày x 7 ngày                  │
│    • Trước ăn 15-30 phút                            │
│    • Tổng: 21 viên                                  │
│    • Giá: 105,000 VND                              │
└─────────────────────────────────────────────────────┘

Tổng cộng: 490,000 VND

Lời dặn:
- Uống thuốc đúng giờ
- Ăn nhẹ nhàng, dễ tiêu
- Tránh ăn cay, chua
- Tái khám nếu không đỡ sau 3 ngày

[Edit]  [Print]  [Send to Pharmacy]  [Sign & Finalize]
```

---

## 5. Clinical Decision Support (CDS)

### 5.1 Drug-Drug Interaction Check

```javascript
// Check for drug interactions
async function checkDrugInteractions(patientId, newDrug) {
  // Get current medications
  const currentMeds = await getActivePatientMedications(patientId);

  const interactions = [];

  for (const currentMed of currentMeds) {
    // Check interaction between new drug and current medications
    const interaction = await drugInteractionDatabase.check(
      newDrug.code,
      currentMed.medication_code
    );

    if (interaction) {
      interactions.push({
        drug1: newDrug.name,
        drug2: currentMed.medication_name,
        severity: interaction.severity, // 'Critical', 'Major', 'Moderate', 'Minor'
        description: interaction.description,
        recommendation: interaction.recommendation
      });
    }
  }

  return interactions;
}
```

**Example interactions:**

| Drug 1 | Drug 2 | Severity | Description |
|--------|--------|----------|-------------|
| Warfarin | Aspirin | Critical | Increased bleeding risk |
| Metformin | Contrast dye | Major | Risk of lactic acidosis |
| ACE inhibitor | K+ supplement | Major | Hyperkalemia risk |
| Amlodipine | Grapefruit | Moderate | Increased drug levels |

**UI Alert:**
```
┌─ ⚠️  CẢNH BÁO TƯƠNG TÁC THUỐC ─────────────────────┐
│                                                      │
│  NGHIÊM TRỌNG (Critical):                           │
│  Warfarin ↔ Aspirin                                 │
│                                                      │
│  Nguy cơ: Tăng nguy cơ chảy máu nghiêm trọng       │
│                                                      │
│  Khuyến nghị:                                        │
│  • Cân nhắc thay thế Aspirin bằng thuốc khác       │
│  • Nếu vẫn phải dùng cả 2, giảm liều và monitor PT/INR│
│  • Giáo dục bệnh nhân về dấu hiệu chảy máu          │
│                                                      │
│  [Xem chi tiết]  [Override & Continue]  [Cancel]    │
└──────────────────────────────────────────────────────┘
```

### 5.2 Drug-Allergy Check

```javascript
async function checkDrugAllergies(patientId, newDrug) {
  const allergies = await getPatientAllergies(patientId);

  for (const allergy of allergies) {
    // Exact match
    if (allergy.allergen_name.toLowerCase() === newDrug.generic_name.toLowerCase()) {
      return {
        alert: true,
        severity: allergy.severity,
        message: `BỆ NH NHÂN DỊ ỨNG ${allergy.allergen_name.toUpperCase()}`,
        reaction: allergy.reaction,
        canOverride: false // Cannot override allergy alert
      };
    }

    // Drug class match (e.g., allergic to Penicillin → alert for Amoxicillin)
    if (isDrugInSameClass(newDrug, allergy.allergen_name)) {
      return {
        alert: true,
        severity: 'High',
        message: `Cảnh báo: ${newDrug.name} cùng nhóm với ${allergy.allergen_name} (bệnh nhân dị ứng)`,
        reaction: allergy.reaction,
        canOverride: true // Can override with justification
      };
    }
  }

  return { alert: false };
}
```

### 5.3 Contraindication Check

```javascript
// Check contraindications based on patient conditions
async function checkContraindications(patientId, newDrug) {
  const medicalHistory = await getPatientMedicalHistory(patientId);

  const contraindications = [];

  // Example: Metformin contraindicated in severe renal impairment
  if (newDrug.generic_name === 'Metformin') {
    const ckd = medicalHistory.find(h => h.icd10_code.startsWith('N18')); // CKD
    if (ckd && ckd.severity === 'Severe') {
      contraindications.push({
        drug: 'Metformin',
        condition: 'Chronic Kidney Disease (Severe)',
        reason: 'Risk of lactic acidosis',
        recommendation: 'Use alternative (e.g., Insulin)'
      });
    }
  }

  // Example: NSAIDs contraindicated in active peptic ulcer
  if (newDrug.therapeutic_class === 'NSAID') {
    const pepticUlcer = medicalHistory.find(h => h.icd10_code === 'K27.0');
    if (pepticUlcer && pepticUlcer.status === 'Active') {
      contraindications.push({
        drug: newDrug.name,
        condition: 'Active Peptic Ulcer',
        reason: 'May worsen ulcer',
        recommendation: 'Use Paracetamol instead'
      });
    }
  }

  return contraindications;
}
```

### 5.4 Duplicate Therapy Check

```javascript
// Check if patient already on similar medication
async function checkDuplicateTherapy(patientId, newDrug) {
  const currentMeds = await getActivePatientMedications(patientId);

  const duplicates = currentMeds.filter(med => {
    // Same drug
    if (med.generic_name === newDrug.generic_name) return true;

    // Same therapeutic class
    if (med.therapeutic_class === newDrug.therapeutic_class) return true;

    return false;
  });

  if (duplicates.length > 0) {
    return {
      alert: true,
      message: `Bệnh nhân đang dùng ${duplicates[0].medication_name} (cùng loại)`,
      recommendation: 'Kiểm tra xem có cần thiết dùng thêm không'
    };
  }

  return { alert: false };
}
```

---

## 6. E-Prescribing

### 6.1 Benefits

- **Accuracy**: No handwriting issues
- **Safety**: Automatic checks (allergies, interactions)
- **Efficiency**: Faster, no transcription errors
- **Tracking**: Know when dispensed
- **Compliance**: Easier to monitor adherence

### 6.2 E-Prescription Format (XML)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Prescription>
  <PrescriptionNumber>RX20231119001</PrescriptionNumber>
  <IssueDate>2023-11-19</IssueDate>
  <Patient>
    <ID>P2023001234</ID>
    <Name>Nguyen Van A</Name>
    <DOB>1985-03-15</DOB>
    <Gender>M</Gender>
  </Patient>
  <Prescriber>
    <Name>Dr. Tran Van C</Name>
    <LicenseNumber>12345</LicenseNumber>
  </Prescriber>
  <Medications>
    <Medication>
      <DrugCode>MED001</DrugCode>
      <DrugName>Paracetamol 500mg Tablet</DrugName>
      <Quantity>21</Quantity>
      <Dosage>1 tablet</Dosage>
      <Frequency>TID</Frequency>
      <Duration>7 days</Duration>
      <Instructions>Take 1 tablet three times daily after meals for 7 days</Instructions>
    </Medication>
    <!-- More medications... -->
  </Medications>
  <Signature>
    <SignedBy>Dr. Tran Van C</SignedBy>
    <SignedAt>2023-11-19T10:30:00</SignedAt>
    <ElectronicSignature>BASE64_ENCODED_SIGNATURE</ElectronicSignature>
  </Signature>
</Prescription>
```

---

## 7. Pharmacy Dispensing

### 7.1 Pharmacist Review Checklist

**Before dispensing:**
- ☐ Verify patient identity
- ☐ Check prescription validity (not expired, properly signed)
- ☐ Verify drug, dose, route, frequency
- ☐ Check for interactions (final safety check)
- ☐ Confirm allergies
- ☐ Check appropriateness (dose appropriate for age/weight/indication)
- ☐ Verify insurance coverage (if applicable)
- ☐ Check inventory availability

### 7.2 Dispensing Screen

```
┌─ DISPENSING - RX#RX20231119001 ────────────────────┐
│  Patient: Nguyễn Văn An (P2023001234)             │
│  Prescriber: Dr. Trần Văn C                        │
└─────────────────────────────────────────────────────┘

Medications to dispense:
┌─────────────────────────────────────────────────────┐
│ ☐ Paracetamol 500mg Tablet                         │
│    Prescribed: 21 tablets                           │
│    In stock: 500 tablets                            │
│    Dispense: [21] tablets                           │
│    Lot#: [LOT20230515] Exp: [05/2025]             │
│                                                     │
│ ☐ Omeprazole 20mg Capsule                          │
│    Prescribed: 28 capsules                          │
│    In stock: 150 capsules                           │
│    Dispense: [28] capsules                          │
│    Lot#: [LOT20230620] Exp: [06/2025]             │
└─────────────────────────────────────────────────────┘

Patient counseling checklist:
☐ Explained how to take each medication
☐ Discussed potential side effects
☐ Reviewed drug interactions
☐ Provided storage instructions
☐ Answered patient questions

Pharmacist signature: [________________]

[Cancel]  [Complete Dispensing]
```

### 7.3 Medication Label

```
┌─────────────────────────────────────────┐
│  PHÒNG KHÁM ABC                        │
│  123 Lê Lợi, Quận 1, TPHCM             │
│  Tel: 028-1234-5678                    │
├─────────────────────────────────────────┤
│  Rx #: RX20231119001                   │
│  Date: 19/11/2023                      │
├─────────────────────────────────────────┤
│  BN: Nguyễn Văn An                     │
│  DOB: 15/03/1985                       │
├─────────────────────────────────────────┤
│  PARACETAMOL 500MG TABLET              │
│                                         │
│  LIỀU DÙNG:                             │
│  Uống 1 viên, ngày 3 lần, sau ăn       │
│                                         │
│  SỐ LƯỢNG: 21 viên                     │
│                                         │
│  Lot #: LOT20230515                    │
│  Exp: 05/2025                          │
├─────────────────────────────────────────┤
│  Pharmacist: Nguyễn Thị D              │
│  Date dispensed: 19/11/2023            │
└─────────────────────────────────────────┘
```

---

## 8. Inventory Management

### 8.1 Stock Control

```sql
CREATE TABLE medication_inventory (
  inventory_id INT PRIMARY KEY AUTO_INCREMENT,
  medication_code VARCHAR(20) NOT NULL,

  -- Stock
  quantity_on_hand DECIMAL(10,2) NOT NULL,
  unit VARCHAR(20),                              -- viên, lọ, tuýp...

  -- Reorder
  reorder_level DECIMAL(10,2),                   -- Minimum stock level
  reorder_quantity DECIMAL(10,2),                -- Auto-order amount

  -- Lot tracking
  lot_number VARCHAR(50),
  expiry_date DATE,
  supplier VARCHAR(255),

  -- Location
  storage_location VARCHAR(100),                 -- Shelf A1, Refrigerator B...

  -- Cost
  unit_cost DECIMAL(10,2),
  total_cost DECIMAL(10,2),

  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,

  FOREIGN KEY (medication_code) REFERENCES medications(medication_code),

  INDEX idx_medication (medication_code),
  INDEX idx_expiry (expiry_date)
);
```

### 8.2 Stock Transactions

```sql
CREATE TABLE inventory_transactions (
  transaction_id INT PRIMARY KEY AUTO_INCREMENT,
  medication_code VARCHAR(20) NOT NULL,

  transaction_type ENUM('Purchase', 'Dispense', 'Return', 'Adjustment', 'Expired'),
  quantity DECIMAL(10,2) NOT NULL,               -- Positive for in, negative for out

  -- Reference
  reference_type VARCHAR(50),                    -- 'Prescription', 'Purchase Order'...
  reference_id INT,                              -- Prescription ID, PO ID...

  -- Details
  lot_number VARCHAR(50),
  transaction_date DATETIME DEFAULT CURRENT_TIMESTAMP,
  performed_by INT,                              -- User ID
  notes TEXT,

  FOREIGN KEY (medication_code) REFERENCES medications(medication_code),
  FOREIGN KEY (performed_by) REFERENCES users(user_id),

  INDEX idx_medication (medication_code),
  INDEX idx_date (transaction_date)
);
```

### 8.3 Automatic Reorder Alert

```javascript
// Check low stock daily
async function checkLowStockMedications() {
  const lowStockItems = await db.query(`
    SELECT medication_code, medication_name,
           quantity_on_hand, reorder_level, reorder_quantity
    FROM medication_inventory
    JOIN medications USING (medication_code)
    WHERE quantity_on_hand <= reorder_level
      AND is_active = TRUE
  `);

  if (lowStockItems.length > 0) {
    // Send alert to pharmacy manager
    await sendAlert('pharmacy_manager', {
      title: 'Low Stock Alert',
      message: `${lowStockItems.length} medications below reorder level`,
      items: lowStockItems
    });

    // Optionally: Auto-generate purchase order
    for (const item of lowStockItems) {
      await createPurchaseOrder(item);
    }
  }
}
```

---

## 9. Medication Adherence & Refills

### 9.1 Medication Adherence Tracking

**For chronic medications (e.g., diabetes, hypertension):**

```javascript
// Calculate adherence rate
function calculateAdherence(prescriptionStartDate, refillDates, daysSupply) {
  const totalDays = daysBetween(prescriptionStartDate, today());
  const daysCovered = refillDates.reduce((sum, refill) => {
    return sum + daysSupply;
  }, 0);

  const adherenceRate = (daysCovered / totalDays) * 100;
  return adherenceRate;
}

// Example:
// Prescription start: 01/01/2023
// Days supply: 30 days per refill
// Refills: 01/01, 05/02, 10/03, 15/04 (4 refills = 120 days supply)
// Total days: 180 days
// Adherence: 120/180 = 66.7%
```

**Adherence categories:**
- Excellent: ≥ 90%
- Good: 80-89%
- Fair: 70-79%
- Poor: < 70%

### 9.2 Prescription Refill Workflow

```
Patient requests refill
    ↓
Check refill eligibility:
- Refills remaining?
- Too early? (based on days supply)
- Prescription expired?
    ↓
If eligible → Process refill
If not → Contact prescriber for new Rx
```

---

## 10. Common Issues & Solutions

### Issue 1: Unclear handwriting (Giấy tay khó đọc)
**Problem**: Pharmacist không đọc được chữ bác sĩ
**Solution**: E-prescribing → eliminate handwriting

### Issue 2: Prescription errors
**Problem**: Sai thuốc, sai liều
**Solution**:
- CDS alerts (real-time checking)
- Standardized order sets
- Pharmacist review

### Issue 3: Non-adherence (Bệnh nhân không uống thuốc đều)
**Problem**: 40-50% chronic disease patients don't take medications as prescribed
**Solution**:
- Patient education
- Medication reminders (app notifications)
- Simplified regimens (reduce frequency if possible)
- Adherence tracking & follow-up

### Issue 4: Drug shortages
**Problem**: Thuốc hết stock
**Solution**:
- Inventory alerts
- Alternative medication suggestions
- Multi-supplier sourcing

---

## Summary

**Key takeaways:**

1. **Safety first**: Allergies, interactions, contraindications checks are MANDATORY
2. **E-prescribing**: Eliminates handwriting errors, enables CDS
3. **SIG codes**: Standard notation (PO, BID, TID...)
4. **Pharmacist review**: Final safety checkpoint
5. **Inventory management**: Track stock, lot numbers, expiry dates
6. **Adherence**: Monitor and support patient compliance

**For implementation:**
- Start with medication catalog (common drugs)
- Implement CDS alerts (critical for safety)
- E-prescribing > paper prescriptions
- Inventory management if internal pharmacy
- Patient education materials
- Mobile app for medication reminders

**Next**: [05-billing-insurance.md](05-billing-insurance.md)
